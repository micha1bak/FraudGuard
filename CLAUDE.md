# [Project name]

[One sentence: what the system does.]

## Working rules

Follow the rules in `docs/README.md` (PRs, branches, commits, Definition of Done, section 1.5 "AI agents"). In short:

- One PR = one task; branch `type/ID-description`, commits in Conventional Commits.
- Every logic change has tests; a bug fix has a test that reproduces the bug.
- Update documentation in the same PR as the code (table in `docs/README.md`, section 3.2).
- Don't merge to `main`, deploy to prod or change scope without a human's approval.
- Don't tick a PR checklist item you haven't verified – say what you didn't check.

## Commands

- Run: `[npm run dev]`
- Tests: `[npm test]`
- Lint: `[npm run lint]`

## Code structure

- `[src/api]` – [contents]
- `[src/web]` – [contents]
- Architecture and domain terms: `docs/architecture.md`, `docs/spec.md` (glossary)

## Conventions

- [e.g. TypeScript strict, no `any`]
- [e.g. names in code match the glossary]

## Don't

- [e.g. don't edit migrations that are already deployed]
- [e.g. don't add dependencies without a justification in the PR]
- Never commit secrets to the repo.
