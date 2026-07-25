# Agentic Payments

**Payment-delegation infrastructure for AI agents: a credential vault plus a scoped authorization layer, so an agent can transact on your behalf without ever holding your card.**

> **Status:** design phase. Sharing the approach and seeking feedback.

## The problem

AI agents can now decide *what* to buy, but there's no safe, standard way for them to *pay*. Every payment rail assumes a human clicking "confirm." If agents are going to transact, someone has to build the delegation and scoped-authorization layer that lets an agent spend within the boundaries a human sets, and nothing more.

## The approach

- **Credential vault** — secrets never touch the agent; it receives scoped, revocable tokens, not your card.
- **Authorization layer** — per-agent spending controls: limits, allowlists, expiry, and human-in-the-loop for anything outside the mandate.

## Docs

- [`STRATEGY.md`](STRATEGY.md) — problem, solution, and integration approach
- [`ARCHITECTURE.md`](ARCHITECTURE.md) — system design and core components
- [`TERMINOLOGY.md`](TERMINOLOGY.md) — payment and protocol concepts

## Related

Part of the broader **Agent Commerce OS** work: see [agent-commerce-os](https://github.com/nikjain15/agent-commerce-os) for the commerce-protocol SDK.

---
MIT © Nik Jain
