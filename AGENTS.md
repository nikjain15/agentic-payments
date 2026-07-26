# Agentic Payments

Payment-delegation infrastructure for AI agents: a credential vault plus a scoped authorization layer, so an agent can transact on your behalf without ever holding your card.

## Project overview

Agentic Payments is a design-stage, docs-only repository. It proposes a reference architecture for how AI agents (such as ChatGPT, Claude, and Perplexity) can pay on a user's behalf within limits the user sets, and nothing more. There is no shipped code here yet. The repository captures the thesis, the system design, the terminology, and the security model, and it is intended for anyone evaluating or contributing to the approach: protocol authors, payment-infrastructure engineers, and reviewers.

## Status and nature of this repository

This is a design and reference repository, not a software project. It contains Markdown design documents only. There is no application code, no package manifest, and therefore no build pipeline or test suite. Everything described in the documents is proposed, not implemented. Please read the documents with that framing.

## Repository structure

Top-level design notes:

- `README.md`: entry point, the problem, the approach, and links to every document.
- `STRATEGY.md`: the problem, the solution, scope, and how the approach fits with the UCP and ACP protocols.
- `ARCHITECTURE.md`: original system design covering core components, data flows, integration points, and the security model.
- `TERMINOLOGY.md`: plain-English reference for payment, authorization, token, and protocol terms.
- `LICENSE`: MIT license.

Extended documentation under `docs/`:

- `docs/PRD.md`: thesis, personas, jobs to be done, success metrics, and a Now/Next/Later roadmap.
- `docs/ARCHITECTURE.md`: reference architecture with delegation-flow and credential-vault trust-boundary diagrams. Extends the top-level `ARCHITECTURE.md`.
- `docs/TECHNICAL_NOTES.md`: a 12-point rubric scored honestly as a design-stage artifact, with the security model as the centerpiece.
- `docs/FDE_JOURNEY.md`: how the system would integrate into a live merchant or PSP environment.
- `docs/EVALS.md`: the evaluations this would require before touching real money.

## Setup, build, and testing

There are no setup, build, or test commands. This repository holds Markdown documents only, so there is nothing to install, compile, or run. To work with it, clone the repository and open the Markdown files in any editor or viewer.

If you want to sanity-check your edits before opening a pull request, preview the Markdown to confirm tables and diagrams render, and check internal links still resolve. No automated gate exists; review is by human reading.

## Documentation conventions

- Write in Markdown. Keep prose plain and direct, matching the existing voice in the documents.
- Frame everything as proposed and design-stage. Nothing here is shipped, so avoid language that implies otherwise.
- Keep terminology consistent with `TERMINOLOGY.md`. Introduce a new term there before using it widely elsewhere.
- Use tables for structured comparisons (components, stakeholders, terms) as the existing documents do.
- Keep ASCII diagrams for flows and architecture, consistent with the current style in `ARCHITECTURE.md`.
- When you add a document under `docs/`, link it from `README.md` so it stays discoverable.
- Keep the two architecture documents aligned: `docs/ARCHITECTURE.md` extends the top-level `ARCHITECTURE.md`, so update both when a design decision changes.

## Contributing and pull request guidelines

Contributing to this repository means editing the design documents, not writing product code. A good contribution sharpens the thesis, corrects the design, clarifies terminology, or strengthens the security reasoning.

- Branch from `main` and use a short, descriptive branch name.
- Keep each pull request focused on one coherent change to the documents.
- Write clear commit messages that describe the intent of the edit.
- In the pull request description, summarize what changed and why, and note any documents that had to move together.
- Because there is no automated test suite, reviewers rely on a careful read. Make edits easy to review by keeping diffs tight and explaining reasoning in the pull request.
- Contributors may use Claude Code or other AI agents as tooling while editing these documents. The documents themselves describe a proposed system and are maintained by hand for correctness.

## Security and secrets

This repository stores no secrets, no credentials, and no environment configuration, because it contains no runnable code. Do not add any. The security discussion here is design-level: it describes how the proposed system would tokenize credentials, encrypt at rest, scope and expire tokens, and log every action for audit. Treat those as design goals to reason about, not as configuration to set.
