# PRD: Agentic Payments (Payment Delegation for AI Agents)

> **Status: design / thesis artifact.** This is a product requirements document for a
> proposed system. Nothing here is shipped. "What's live" is honest: the repository
> is a set of design documents. Everything in the Roadmap is framed as intent, not fact.

---

## 1. Thesis

AI agents can now decide *what* to buy. They cannot safely *pay*. Every payment rail in
production assumes a human is present at the moment of confirmation: a person clicks "Pay,"
re-enters a CVV, approves a 3-D Secure challenge, or taps a wallet. When you remove the human
from that final step, you are left with two bad options:

1. **Give the agent raw credentials** (a card number, a wallet key, a stored-card token with
   no constraints). This makes every agent a full-privilege spender and a fresh breach surface.
2. **Keep a human in every loop.** This defeats the point of delegating the task in the first place.

The missing layer is **scoped, revocable payment delegation**: a way for a human to hand an
agent a *mandate* ("you may spend up to $200 at these merchant categories this week, single
purchases under $50 auto-approve, anything larger asks me") without ever handing over the
underlying secret. The agent holds a constrained, expiring, auditable token. The human holds
the controls.

This is the same problem OAuth solved for API access a decade ago, applied to money. OAuth let
you grant an app scoped access to your data without sharing your password. Agentic Payments
proposes the equivalent for spend: **grant an agent scoped access to your money without sharing
your card.**

**One-line framing:** *OAuth for spend. The agent gets a scoped, revocable mandate, never the card.*

---

## 2. Where this sits (and what it is not)

Emerging agent-commerce protocols (ACP, the Agentic Commerce Protocol; UCP, a Universal
Commerce Protocol pattern) define how an agent and a merchant run a *checkout*: discover
products, build a cart, complete an order. They deliberately do **not** specify how a user
authorizes an agent to reach their payment credentials in the first place. They assume the
agent "has" a payment method and move on.

That assumption is the gap this design fills.

| This IS | This is NOT |
|---|---|
| A delegation + authorization layer (the "who may spend, how much, where" decision) | A payment processor (Stripe, Adyen already charge the card) |
| A credential vault that returns scoped tokens, never raw cards | A shopping/checkout experience (agents and merchants own that) |
| A user consent and revocation surface | A fraud/risk engine (PSPs own scoring) |
| A reference implementation of the missing consent layer for ACP/UCP | A merchant-of-record or a wallet |

Related work in the same portfolio: **agent-commerce-os** (the commerce-protocol SDK, the
"how do agent and merchant transact" layer). Agentic Payments is the **authorization and
credential-custody** layer beneath it. See [ARCHITECTURE.md](ARCHITECTURE.md) for how the two compose.

---

## 3. Personas and Jobs-to-be-Done

### Persona A: The delegating user ("the principal")
A person who wants an agent to run a real errand that costs money: restock household goods,
book travel under a cap, buy a specific part, subscribe to a tool.

- **JTBD:** *"When I hand my agent a task that involves spending, I want to set hard limits it
  cannot exceed, so that I can walk away without fear of a runaway charge or a leaked card."*
- **Success looks like:** one-time setup, per-agent controls, a single dashboard showing every
  agent's spend, and a revoke button that works in one click.

### Persona B: The agent platform ("the integrator")
An assistant/agent builder (a ChatGPT-class app, a vertical agent, an in-house workflow agent)
that needs to complete purchases without becoming a payments company.

- **JTBD:** *"When my agent needs to pay, I want to call one authorization API and receive a
  merchant-bound token, so that I never store card data or carry PCI liability."*
- **Success looks like:** a single `authorize-payment` call, tokens that drop straight into
  ACP/UCP checkout, and a clean failure mode when the request exceeds the user's mandate.

### Persona C: The merchant / PSP ("the acceptor")
The party that ultimately charges the card and ships the goods.

- **JTBD:** *"When an agent presents payment, I want cryptographic evidence that a real user
  authorized this specific spend, so that I can accept it with the same or lower fraud risk as
  a human checkout."*
- **Success looks like:** a verifiable authorization artifact (who authorized, what mandate,
  what constraints), settlement through existing rails, and a clear audit trail if disputed.

### Persona D (secondary): The compliance / security reviewer
The person at the integrator or merchant who has to sign off.

- **JTBD:** *"Before we let agents spend, I want least-privilege by construction, instant
  revocation, and a complete immutable audit log, so that I can bound the blast radius of a
  compromised agent."*

---

## 4. Success Metrics (what this WOULD be measured on)

These are proposed targets for a future implementation, not measured results.

| Category | Metric | Target intent |
|---|---|---|
| **Safety (north star)** | Unauthorized-spend rate: value of charges outside a user's mandate ÷ total delegated value | Drive toward **zero**; this is the whole product |
| Safety | Revocation propagation time (revoke click → token unusable) | < 1 second, hard-enforced server-side |
| Safety | Blast radius: max spend a fully compromised agent token can move before limits stop it | Bounded by mandate, never "the whole card" |
| Trust / UX | Delegation setup completion rate | High; setup is one-time |
| Trust / UX | Human-in-the-loop escalations that resolve without abandonment | High; escalation should feel like a nudge, not a wall |
| Integrator | Time-to-first-authorized-payment for a new integrator | Hours, one API |
| Integrator | Card-data-touched by integrators | **Zero** (they only ever see scoped tokens) |
| Acceptance | Agent-checkout authorization-acceptance rate vs. human-checkout baseline | At or above human baseline |
| Auditability | % of transactions with a complete, tamper-evident authorization chain | 100% |

---

## 5. Non-goals

- Building a PSP, wallet, or merchant-of-record.
- Fraud scoring, KYC, or risk analytics (consumed from PSPs, not rebuilt).
- A general budgeting/personal-finance app. The dashboard is a control surface, not Mint.
- Replacing ACP or UCP. This *completes* them.

---

## 6. Key product tradeoffs

1. **Friction vs. safety.** Every additional human confirmation is safer and slower. The design
   resolves this with *mandates*: the human sets the rules once, and only out-of-mandate events
   escalate. The tuning knob (what auto-approves vs. what asks) is the core UX bet.
2. **Token lifetime vs. usability.** Short-lived, single-use, merchant-bound tokens are safest
   but break recurring/subscription spend. The design keeps single-use as the default and treats
   multi-use tokens as a separate, tightly constrained type (see [TERMINOLOGY](../TERMINOLOGY.md)).
3. **Custody vs. liability.** Holding tokenized credentials centralizes value and invites PCI
   scope and breach risk. The design leans on PSP tokenization (never storing raw PANs) so the
   vault holds *references*, not card numbers. The trust boundary is the security centerpiece
   (see [TECHNICAL_NOTES](TECHNICAL_NOTES.md)).
4. **Standard-fit vs. differentiation.** Fitting cleanly into ACP/UCP tokenization handlers wins
   adoption but constrains the token format. The design chooses standards-fit first.

---

## 7. Roadmap, Now / Next / Later

**Now (this repository, design stage)**
- Thesis, problem framing, and positioning (this PRD).
- Reference architecture with delegation flow and credential-vault trust boundary
  ([ARCHITECTURE.md](ARCHITECTURE.md)).
- Security model: least-privilege, scoped/delegated tokens, revocation, audit
  ([TECHNICAL_NOTES.md](TECHNICAL_NOTES.md)).
- Integration thesis for a merchant/PSP environment ([FDE_JOURNEY.md](FDE_JOURNEY.md)).

**Next (a first buildable slice, if pursued)**
- A minimal authorization engine spec: mandate schema, limit evaluation, deterministic
  allow/deny with a machine-readable reason.
- A token-issuance spec: single-use, merchant+amount-bound, short TTL, signed.
- A reference OAuth-style consent flow and a revocation endpoint with server-side enforcement.
- An audit-log spec: append-only, tamper-evident, one entry per authorization decision.
- The eval plan this would require before touching real money ([EVALS.md](EVALS.md)).

**Later (production hardening)**
- ACP `delegate_payment` and UCP tokenization-handler reference adapters.
- Multi-use/subscription mandates with independent, revocable schedules.
- Anomaly detection over the audit log (spend that fits the mandate but not the pattern).
- Formal threat model review and third-party security assessment before any live pilot.

---

## 8. Open questions (honest gaps)

- **Mandate expressiveness.** How rich should limits be (category, merchant, velocity, time-of-day)
  before the policy language becomes its own attack surface and support burden?
- **Dispute and chargeback flow.** When an agent buys the wrong thing within its mandate, who owns
  the reversal, and what evidence does the authorization chain provide?
- **Cross-agent conflicts.** If two agents share one funding source, how are limits reserved and
  reconciled to prevent double-spend against the same daily cap?
- **Regulatory surface.** Custody of payment credentials and spend authorization may trigger
  money-transmission, PCI, and PSD2/SCA-style considerations that a design doc cannot resolve and
  that would need counsel before a pilot. **[VERIFY WITH NIK]** whether any regulatory scoping exists.
