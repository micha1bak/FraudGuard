# Project documentation guide

Version 1.0 · 2026-11-26

## 0. Project facts

| Field | Value |
| --- | --- |
| Project | FraudGuard – checks bank transfers for fraud risk before they are sent, warns manipulated customers and routes suspicious transfers to human review |
| Requirements | [REQUIREMENTS.md](../REQUIREMENTS.md), [spec.md](spec.md) |
| Repository | [local only – no remote yet] |
| Tracker | [not set up yet – GitHub Issues / Jira] |
| Stack | [not chosen yet – will be recorded in `decisions.md`] |
| Environments | [not defined yet – dev / staging / prod] |

## 1. Working rules

Every change follows the same path: **task → branch → PR → merge to `main`**. Nothing reaches `main` without a PR.

### 1.1 Pull requests

1. **One PR = one task.** Title: `[ID] Short description`, e.g. `[PROJ-12] Sign in with Google`.
2. **Tests are mandatory.** New logic comes with tests; a bug fix includes a test that reproduces the bug.
3. **Documentation in the same PR.** If the change touches anything in the table in 3.2, the PR updates the matching file.
4. **Small PRs.** Aim for about 400 changed lines at most; split larger tasks.
5. **Green CI before merge.**
6. **A checklist instead of formal review.** Before merging, the author goes through the checklist in the template and ticks it honestly. A review by a second person is welcome but not required.

PR description template (`.github/pull_request_template.md`):

```markdown
## Task
[ID](link) – what and why

## Changes
- ...

## Checklist
- [ ] the PR covers one task
- [ ] tests added / updated, CI green
- [ ] I reviewed my own diff (no debug code, secrets or dead code)
- [ ] documentation updated (or: not applicable, because ...)
- [ ] decision added to decisions.md (if there was one)

## Risks
- migrations, configuration changes, how to roll back
```

### 1.2 Branches, commits, versions

- Branch: `type/ID-description`, types: `feature`, `fix`, `refactor`, `docs`, `chore`. Example: `fix/PROJ-30-empty-cart`.
- Commits: Conventional Commits, e.g. `feat(auth): sign in with Google`.
- Versions: SemVer (`v1.4.0`) as a tag on `main`; every release adds an entry to `CHANGELOG.md`.

### 1.3 Tests and CI

A merge is only possible when these pass: **build, linter/formatter, tests, dependency scan** (no critical vulnerabilities). Test coverage must not drop compared with `main`.

### 1.4 Definition of Done

A task is done when:

- [ ] the acceptance criteria are met and covered by tests,
- [ ] the PR is merged and CI is green,
- [ ] the documentation is up to date,
- [ ] the change works in [staging / prod].

### 1.5 AI agents

An agent (e.g. Claude Code) is treated like any PR author and follows the same rules. In addition:

- **The agent may on its own:** create branches, write code and tests, update documentation, open PRs and fill in the checklist.
- **Requires a human:** merging to `main`, deploying to prod, changing scope or requirements, adding a dependency with wide impact, operations on production data and secrets.
- The agent reads `CLAUDE.md` before starting and updates documentation in the same PR as the code.
- The agent does not tick a checklist item it has not verified; instead it states what it did not check.

## 2. Requirements and specification

Requirements live on two levels:

| Level | Where | Contents |
| --- | --- | --- |
| Project specification | repo: `/docs/spec.md` | goal, scope, what we don't do, non-functional requirements – changes rarely |
| User stories / tasks | tracker | concrete features to build – change often |

### 2.1 Project specification (`/docs/spec.md`)

```markdown
# Specification: [name]

## Goal
What problem we solve and for whom (2–3 sentences).

## Users
- [role] – what they need

## Scope
- F-01 [main feature]
- F-02 ...

## Out of scope
- what we deliberately don't do

## Non-functional requirements
Each one measurable, e.g.:
- NFR-01 Performance: [API p95 response < 300 ms]
- NFR-02 Security: [no critical vulnerabilities in the scan]
- NFR-03 Accessibility: [WCAG 2.2 AA]
```

### 2.2 User story (in the tracker)

```markdown
As a [role]
I want [action]
so that [benefit].

Acceptance criteria:
- [ ] [verifiable condition – main scenario]
- [ ] [error / edge case]

Related: F-0X, NFR-0X
```

Rules:

- A story fits in one or a few PRs; split anything larger.
- Every acceptance criterion can be checked by a test or manually in a few steps.
- A task in the tracker points to its story, and the story to a feature in the specification (`F-0X`).
- A change of scope = an update to `spec.md` in a PR.

## 3. Documentation and keeping it current

Documentation lives in the repo next to the code and changes in the same PR as the code. Outdated documentation is a bug.

### 3.1 Structure

```text
README.md            ← what the project is, how to run it, how to test it
CLAUDE.md            ← rules for AI agents
CHANGELOG.md         ← release history
.github/pull_request_template.md  ← PR template
/docs
  README.md          ← this guide (working rules)
  spec.md            ← project specification
  architecture.md    ← architecture: C4 model + Mermaid diagrams
  decisions.md       ← decision log
```

### 3.2 What to update when

| Change | Update |
| --- | --- |
| how to run, configuration, environment variables | `README.md` |
| scope, new feature, non-functional requirement | `docs/spec.md` |
| new module, service, database, integration, changed flow | `docs/architecture.md` (+ diagram) |
| technology choice, trade-off, deliberate technical debt | `docs/decisions.md` |
| new convention or rule the agent must know | `CLAUDE.md` |
| release | `CHANGELOG.md` |

### 3.3 Architecture (`/docs/architecture.md`)

We describe the architecture with the **C4 model** and write diagrams in Mermaid:

| C4 level | Question | Required? |
| --- | --- | --- |
| 1. System context | who uses the system and what does it integrate with? | yes |
| 2. Containers | which applications, services and databases make it up and how do they communicate? | yes |
| 3. Components | how is a complex container built inside? | only when needed |
| 4. Code | – | not drawn; the code is the source of truth |

On top of that: a **sequence diagram** for 1–3 key flows, a short deployment table and cross-cutting concepts (authentication, logging, personal data). Add more diagrams only when they answer a question that text cannot. Names on diagrams match the code and the glossary (`spec.md`).

### 3.4 Decision log (`/docs/decisions.md`)

One decision = one short entry, newest first. Entries are not edited; changing a decision means a new entry that points to the previous one.

```markdown
## D-003: [Decision title] – 2026-11-26
- Context: the problem and constraints
- Decision: we choose ..., because ...
- Rejected: option B – why not
- Consequences: what we gain, what we lose
- Supersedes: D-001 (if applicable)
```

### 3.5 AI agent file (`CLAUDE.md`)

Short (about one page), concrete, always current:

```markdown
# [project name]
- Working rules: docs/README.md (PRs, branches, DoD, section 1.5)
- Run: [command]   Tests: [command]   Lint: [command]
- Code structure: [directory] – [contents]
- Conventions: [naming, style, patterns we use]
- Don't: [e.g. don't change migrations after deployment, don't add dependencies without asking]
```
