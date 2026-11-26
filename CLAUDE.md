# FraudGuard

Checks every bank transfer for fraud risk before it is sent, warns customers who may be manipulated by a scammer, and routes suspicious transfers to a bank employee for review.

Requirements: `REQUIREMENTS.md`. Scope, features (F-0X), non-functional requirements (NFR-0X) and glossary: `docs/spec.md`.

## Working rules

Follow the rules in `docs/README.md` (PRs, branches, commits, Definition of Done, section 1.5 "AI agents"). In short:

- One PR = one task; branch `type/ID-description`, commits in Conventional Commits.
- Every logic change has tests; a bug fix has a test that reproduces the bug.
- Update documentation in the same PR as the code (table in `docs/README.md`, section 3.2).
- Don't merge to `main`, deploy to prod or change scope without a human's approval.
- Don't tick a PR checklist item you haven't verified – say what you didn't check.
- Don't commit unless a human explicitly asks for a commit.

## Commands

The technology stack has not been chosen yet (see `docs/decisions.md`). Fill in once the build setup exists.

- Run: `[TBD]`
- Tests: `[TBD]`
- Lint: `[TBD]`

## Code structure

- `src/` – source code (empty; structure to be defined together with the stack)
- Architecture and domain terms: `docs/architecture.md`, `docs/spec.md` (glossary)

## Conventions

- Names in code match the glossary in `docs/spec.md` (e.g. `HeldTransfer`, `RiskScenario`, `RiskThreshold`, `ReviewTimeLimit`, `AuditLog`).
- Link user stories, PRs and tests to features and requirements by ID (`F-0X`, `NFR-0X`).
- [language and style conventions – TBD with the stack]

## Don't

These come from the regulatory and security requirements (`docs/spec.md`, section 5) – treat them as hard rules:

- Never send data that could identify the customer to an external AI service (NFR-03). Anonymise before the call and cover it with tests.
- Never let the system approve or reject a held transfer on its own – the final decision is always a person's, with a justification (F-08).
- Never update or delete audit log entries; the audit log is append-only (NFR-01).
- Never make transfer processing depend on the AI service being available; there must always be a fallback (NFR-04).
- Never execute a held transfer after its review time limit has passed (F-09).
- Never use real customer data in tests, fixtures or logs.
- Don't add dependencies without a justification in the PR.
- Never commit secrets to the repo.
