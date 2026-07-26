# Technical Notes: Rubric Scorecard (Design-Stage)

> **Status: design / thesis artifact, scored honestly.** This repository contains design
> documents, not code. The 12-point rubric below is scored for a *design-stage* artifact: a high
> score means the design *reasoning* is strong and explicit, not that anything is built. Where the
> rubric asks about implementation (running code, live evals, deployed guardrails), the honest
> score is low and marked as roadmap. The security model is the centerpiece and is scored hardest.

---

## 1. Scorecard

| # | Dimension | Score (0-5) | Evidence | Gap |
|---|---|---|---|---|
| 1 | Model choice (LLM vs ML vs hybrid) | 3.0 | Correctly identifies that the core authorization decision must be **deterministic policy**, not an LLM. An LLM's role is bounded to the agent *above* this layer (deciding what to buy) and optionally to anomaly explanation over the audit log. Money-moving decisions are rules, not generation. | No decision matrix yet for where (if anywhere) an ML model scores anomaly risk vs. deferring entirely to PSP fraud signals. |
| 2 | How the AI works (grounding, context) | 3.0 | The design keeps the LLM out of the trust boundary: the agent is untrusted and its output is treated as a *request*, never an instruction the engine obeys. Every request is re-grounded against the user's mandate at spend time. | No prompt/context spec, because there is intentionally no LLM inside the authorization path yet. Anomaly-explanation grounding is roadmap. |
| 3 | Tools / MCP (schemas, validation, errors) | 3.5 | Single narrow public surface (`authorize-payment`) with a clear contract: bounded inputs (grant, merchant, amount, context), and **structured denials** (machine-readable reason) rather than free-text errors. Least-surface-area is an explicit design choice. | No published JSON schema, no versioning story, no idempotency-key spec for retried requests yet. |
| 4 | Agents & skills | 2.5 | Clear separation of concerns: the agent proposes, the engine disposes. The agent holds a grant that is *not spendable alone*. | This repo does not implement an agent; it defines the layer an agent calls. Fine for scope, but nothing to run. |
| 5 | Orchestration & routing (multi-model, cost) | 2.0 | Correctly out of scope for the authorization layer: routing lives in the agent platform above. The design avoids over-building. | No cost model for token issuance at scale, no throughput/latency budget for the engine. |
| 6 | RAG & context (retrieval, failure modes) | 2.5 | The authorization engine "retrieves" the user's mandate and the vault reference deterministically (a lookup, not a vector search), appropriate, since guessing is unacceptable when money moves. Audit-log retrieval powers the dashboard. | Anomaly detection over the audit log (the one place retrieval/embeddings could help) is roadmap only. |
| 7 | Evals & grounding | 2.0 | [EVALS.md](EVALS.md) lays out the eval ladder this *would* need (deterministic policy unit tests → adversarial mandate-bypass suite → red-team → shadow-mode A/B) with named metrics (unauthorized-spend rate, false-deny rate, revocation-propagation latency). | Nothing implemented. Explicitly a roadmap, not a harness. |
| 8 | Code quality | 1.5 | No code, so no code quality to score. Design docs are structured and internally consistent. | The top-level `STRATEGY.md` has visible copy-paste artifacts (duplicated "Integration with Protocols" sections, a broken markdown heading `###Impact`). Flagged for cleanup. |
| 9 | Scalability & cost | 2.5 | Stateless authorization evaluation and short-lived tokens scale horizontally; the vault is the one stateful, isolated bottleneck by design. | No quantified targets (requests/sec, token-mint latency, vault QPS), no cost-per-authorization estimate. |
| 10 | Guardrails & safety | 4.0 | **Centerpiece, and the strongest dimension.** Least-privilege by construction, default-deny engine, secret never leaves the vault, single-use merchant+amount-bound short-TTL tokens, instant server-side revocation, append-only tamper-evident audit, human-in-the-loop for out-of-mandate spend, and a bounded breach blast radius (vault holds references, not PANs). See section 3. | Not yet threat-modeled formally; token signing, replay defense, and key management are described in principle but not specified. |
| 11 | Product layer (PRD) | 4.0 | [PRD.md](PRD.md) has crisp personas, JTBD, a safety north-star metric (unauthorized-spend rate), explicit non-goals, four named tradeoffs, and a Now/Next/Later roadmap. Positioning ("OAuth for spend") is sharp and correctly scoped against ACP/UCP. | Success metrics are targets, not measured; regulatory scoping is an open question. |
| 12 | FDE journey | 3.5 | [FDE_JOURNEY.md](FDE_JOURNEY.md) walks a realistic integration into a merchant/PSP environment: integration points, secrets handling, shadow/parallel rollout, observability, and de-risking, drawn from a real modernization playbook. | Hypothetical (no customer), so cutover specifics are illustrative. |

**Design-stage weighted read:** strong on **problem framing, product thesis, and the security
model** (the three that matter most for a design artifact about money delegation); honestly weak on
**anything requiring running code** (evals, code quality, measured scale). That distribution is
correct for what this is: a thesis, not a product.

---

## 2. Model and decision architecture

The single most important architectural claim: **the authorization decision is deterministic
policy evaluation, not an LLM call.** When money moves, "usually right" is a failure. The engine is
a rules evaluator over the user's mandate with a default-deny posture. The LLM lives strictly
*above* the trust boundary, in the agent that decides what to purchase, and its output is treated as
an untrusted *request*. The one place ML could legitimately enter is anomaly detection over the
audit log (spend that fits the mandate but not the historical pattern), and that is explicitly
roadmap, not core.

---

## 3. Security model (the centerpiece)

Scored 4.0/5 as a design because the reasoning is explicit and correct; not 5 because none of it is
threat-modeled or implemented.

- **Least privilege by construction.** A token can only ever do what the mandate allows. Default is
  deny. There is no "full access" grant to misuse.
- **Scoped, delegated tokens.** Payment tokens are single-use, bound to a specific merchant and
  amount, short TTL, and signed. A leaked token is worth one bounded purchase, once, at one merchant,
  for a short window, not a card.
- **Secret custody.** Raw PANs never enter the system; the vault holds PSP tokenization references,
  encrypted at rest, in the highest-trust isolated zone. A vault breach yields references, not
  spendable numbers: bounded blast radius.
- **Revocation.** Server-side and instant. Revoking a grant invalidates outstanding tokens rather
  than trusting the agent to comply. Revocation-propagation latency is a named eval metric.
- **Audit.** Append-only, tamper-evident, one entry per authorization decision (allow, deny, escalate)
  with the reason. This is both the user-visibility surface and the dispute-evidence chain.
- **Human-in-the-loop.** Anything outside the mandate escalates to a per-transaction approval; the
  human is not in every loop, only the ones that exceed what they pre-authorized.

**Named residual risks (honest):** token signing-key management and rotation are unspecified; replay
defense for single-use tokens is asserted but not designed; the mandate policy language is itself an
attack/complexity surface; and custody plus spend-authorization likely carry PCI and
money-transmission regulatory weight that a design doc cannot resolve. These are the first things a
security review would open. **[VERIFY WITH NIK]**: whether any formal threat model or regulatory
scoping already exists outside this repo.

---

## 4. Cost and scale (design intent)

Authorization evaluation is stateless and horizontally scalable; tokens are short-lived so there is
no large token store to manage; the credential vault is the single isolated stateful component and
the natural scaling and hardening focus. No throughput, latency, or cost-per-authorization numbers
are claimed, because none have been measured.

---

## 5. Honest gaps summary

1. No running code, tests, or evals, everything in [EVALS.md](EVALS.md) is a plan.
2. Security model is sound in principle but not threat-modeled; key management and replay defense
   are unspecified.
3. Top-level `STRATEGY.md` has copy-paste and heading artifacts worth cleaning up (does not affect
   the design's substance).
4. Regulatory and dispute/chargeback flows are open questions, not designs.
