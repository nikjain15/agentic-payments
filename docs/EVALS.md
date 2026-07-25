# Evaluation Plan (Roadmap)

> **Status: roadmap, not a harness.** Nothing here is implemented. This document describes the
> evaluation ladder a system that authorizes real spend *would need* before it ever touches money,
> and the metrics it would be judged on. It is included so the design is honest about what
> "trustworthy" would require, not to claim any of it exists.

For a payment-authorization layer, evals are not a nicety. The product's north star is a *safety*
metric (unauthorized spend), so the eval suite is closer to a security test plan than a model
benchmark. The core decision is deterministic policy, so the base of the ladder is ordinary unit
testing, not LLM judging.

---

## 1. The eval ladder

### Tier 1 - Deterministic policy unit tests (the foundation)
The authorization engine is rules, so it must be exhaustively unit-tested like rules.

- Every mandate dimension (per-transaction cap, daily cap, velocity, merchant allowlist, category,
  expiry) tested at and across its boundary.
- Default-deny verified: any request not explicitly permitted is denied.
- Structured-denial correctness: the machine-readable reason matches the rule that fired.
- **Metrics:** branch/decision coverage of the policy evaluator (target near-complete);
  zero known allow-when-should-deny cases.

### Tier 2 - Adversarial mandate-bypass suite
Treat the agent as hostile and try to spend outside the mandate.

- Amount just over cap, split purchases to evade a daily cap, replay of a single-use token,
  expired-token reuse, merchant substitution on a bound token, race conditions against a shared cap.
- Revocation races: spend attempted in the window immediately after a revoke.
- **Metrics:** unauthorized-spend rate under adversarial load (**target zero**); replay-success rate
  (target zero); revocation-propagation latency distribution.

### Tier 3 - LLM-judge (only above the trust boundary)
The LLM is not in the authorization path, so LLM-judging applies to the *agent's* behavior and to
anomaly explanations, never to allow/deny decisions.

- Judge whether an anomaly explanation over the audit log is faithful to the underlying events
  (grounding / no fabrication).
- Judge whether agent-facing denial messages are actionable without leaking policy internals.
- **Metrics:** explanation-faithfulness rate; leak rate of sensitive mandate detail.

### Tier 4 - Red-team / security review
Human adversarial testing plus formal threat modeling of token signing, key management, and the
mandate policy language as an attack surface. This is a gate, not a metric: no live pilot without it.

### Tier 5 - Shadow-mode and A/B in a live environment
As described in [FDE_JOURNEY.md](FDE_JOURNEY.md): evaluate decisions with no value moving, then a
capped parallel run.

- **A/B / comparison metrics:** authorization-acceptance rate vs. human-checkout baseline;
  false-deny rate (legitimate agent spend wrongly blocked, the usability cost of safety);
  escalation-resolution rate; end-to-end unauthorized-spend rate in production.

---

## 2. Named metrics summary

| Metric | Definition | Target intent |
|---|---|---|
| Unauthorized-spend rate | Value charged outside a mandate ÷ total delegated value | Zero (north star) |
| Replay-success rate | Single-use tokens successfully reused | Zero |
| Revocation-propagation latency | Revoke click → token unusable | Bounded, sub-second |
| False-deny rate | Legitimate in-mandate spend wrongly blocked | Low (usability cost of safety) |
| Policy evaluator coverage | Decision coverage of the rules engine | Near-complete |
| Explanation-faithfulness | Anomaly/denial explanations grounded in real events | High |
| Acceptance rate vs. human baseline | Agent-checkout authorizations accepted vs. human checkout | At or above baseline |

---

## 3. What is implemented today

Nothing. This is a design artifact. The first buildable slice ([PRD.md](PRD.md), "Next") would start
at Tier 1 - a deterministic policy evaluator with an exhaustive unit suite and an adversarial
bypass suite - because for a system that moves money, that base tier is the minimum bar before any
token is ever issued for real value.
