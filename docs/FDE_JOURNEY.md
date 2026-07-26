# FDE Journey: Landing Agentic Payments in a Live Merchant / PSP Environment

> **Status: design / thesis artifact.** This is a hypothetical forward-deployed integration
> narrative for a system that is not yet built. There is no customer and no deployment. It exists
> to show *how* this design would deploy into a production payments environment, what would break,
> and how the rollout would be de-risked. The de-risking method is drawn from a real large-scale
> financial-modernization playbook: shadow/parallel running, exact-parity reconciliation, and
> staged cutover with sign-off before real value moves.

---

## 1. The environment we would deploy into

Assume a mid-to-large merchant (or a platform serving many merchants) that already:

- Charges cards through an existing PSP (Stripe, Adyen, or similar) as merchant-of-record.
- Runs a checkout that expects a human, or is piloting an agent-commerce protocol (ACP/UCP).
- Has a security/compliance function that must sign off before anything touches payment credentials.
- Has a fraud/risk stack it trusts and does not want to replace.

Agentic Payments does **not** replace any of that. It slots in as the **authorization and
credential-delegation layer** so agents can pay through the *existing* rails within *user-set*
limits. The PSP still charges the card; the merchant still ships; fraud scoring is unchanged.

---

## 2. Integration points

| Integration point | What connects | Ownership |
|---|---|---|
| **PSP tokenization** | The vault stores PSP tokenization references, so raw PANs never enter our system | PSP keeps card custody; we hold references |
| **Checkout / protocol** | Tokens drop into ACP `delegate_payment` or a UCP tokenization handler | Merchant/platform owns checkout; we supply the pre-authorized token |
| **Agent platform** | The agent calls one `authorize-payment` endpoint and receives a scoped token or a structured denial | Agent platform integrates one API; carries no card data |
| **Identity / consent** | OAuth-style grant/revoke flow ties an agent to a user's mandate | We own the consent surface; the merchant consumes the resulting authorization artifact |
| **Audit / SIEM** | Append-only decision log streams to the merchant's observability and compliance tooling | Shared: we produce the chain, they retain and monitor it |

The integration surface is deliberately small: one outbound API for agents, one PSP reference
relationship, one protocol adapter, one consent flow, one audit stream.

---

## 3. Security and secrets handling at deploy time

- **No raw card data crosses to us.** We integrate with the PSP's tokenization, so the merchant's
  PCI scope does not expand into our system in the way storing PANs would.
- **Least-privilege service credentials.** The integration runs with narrowly scoped keys; the
  agent-facing API can only mint bounded tokens, never read vault contents.
- **Key management up front.** Token-signing keys, rotation, and the vault's encryption keys are
  provisioned and reviewed before any pilot. (This is named as an unspecified gap in
  [TECHNICAL_NOTES.md](TECHNICAL_NOTES.md) and would be closed during onboarding, not assumed.)
- **Revocation tested first.** Before real spend, we prove that a revoked grant makes outstanding
  tokens unusable server-side, with measured propagation latency.

---

## 4. Rollout and cutover (borrowed from a real modernization playbook)

Money-moving systems are not switched on with a flag flip. The rollout mirrors how a
trillion-dollar platform migration is de-risked: **run the new alongside the old until values match
exactly, then cut over with sign-off.**

1. **Shadow mode (no value moves).** The authorization engine evaluates real (or replayed) agent
   requests and records allow/deny/escalate decisions to the audit log, but issues no spendable
   tokens. We compare its decisions against expectation.
2. **Parallel run with hard caps.** Enable real token issuance for a small cohort under conservative
   mandates (low per-transaction and daily caps, tight merchant allowlists). Every authorized charge
   is reconciled against the mandate that produced it.
3. **Parity and reconciliation.** For every transaction: did the charge fall inside the user's
   mandate? Is there exactly one immutable audit entry with a correct reason? Does revocation stop
   spend within budget? Any discrepancy is traced to root cause and fixed before scope widens.
4. **Staged cutover.** Widen the cohort and raise caps only after clean parallel-run results and a
   security-review sign-off. The human-in-the-loop escalation path stays on throughout.
5. **Rollback ready.** A single revoke-all switch disables issuance instantly; because tokens are
   short-lived and single-use, the in-flight blast radius of a rollback is naturally bounded.

The gate to widen or cut over is the same three-part test used in serious modernizations:
**behavioral parity + time-in-parallel + explicit human sign-off.**

---

## 5. Observability

- **Decision-level telemetry.** Every authorization emits a structured event (allow/deny/escalate,
  reason, mandate id, merchant, amount) to the merchant's SIEM and to the user dashboard.
- **Safety SLOs.** Track the north-star unauthorized-spend rate (target zero), false-deny rate
  (legit agent spend wrongly blocked), revocation-propagation latency, and escalation resolution.
- **Anomaly signals.** Spend that fits the mandate but not the historical pattern is flagged for
  review (roadmap; see [EVALS.md](EVALS.md)).

---

## 6. De-risking summary

| Risk | Mitigation |
|---|---|
| Runaway or compromised agent | Mandate caps + default-deny + short-lived single-use bound tokens bound the blast radius |
| Credential breach | Vault holds PSP references, not PANs; a breach yields non-spendable references |
| "Silent" wrong authorizations | Shadow mode + parallel reconciliation catch decision errors before value moves |
| Revocation that does not take | Revocation is server-side, tested pre-pilot, with measured propagation latency |
| Disputes | Append-only tamper-evident audit chain provides per-transaction authorization evidence |
| Regulatory exposure | Scoped before pilot with counsel; **[VERIFY WITH NIK]** whether scoping exists |

---

## 7. What a first engagement would actually deliver

A realistic first forward-deployed slice: one merchant, one PSP, one agent platform, a low-cap
mandate for an internal cohort, shadow mode into a tightly-capped parallel run, full audit streaming
to their tooling, and a documented parity/sign-off gate before any expansion. The deliverable is not
"agents can now spend freely", it is *"agents can spend exactly within the limits a human set, with
proof, and we can turn it off in one click."*
