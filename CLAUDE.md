# FraudGuard

Checks every bank transfer for fraud risk before it is sent, warns customers who may be manipulated by a scammer, and routes suspicious transfers to a bank employee for review.

Standalone proof of concept (PoC): no integration with bank systems. Accounts, transfers and users are simulated with test data, and the only external service is the OpenAI API.

This repo is the **backend**: Java 25, Spring Boot, Maven, PostgreSQL, REST/JSON API. The React frontend is in a separate repo. Stack decision: `docs/decisions.md` (D-001).

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

Planned – the Maven project and `docker-compose.yml` don't exist yet. Confirm and update these once they do.

- Start database: `docker compose up -d db`
- Run API: `./mvnw spring-boot:run`
- Unit tests: `./mvnw test`
- All tests (incl. integration on PostgreSQL via Testcontainers): `./mvnw verify`
- Format code: `./mvnw spotless:apply` (check: `./mvnw spotless:check`, also part of `verify`)

## Code structure

Planned standard Maven layout (see `docs/architecture.md`, sections 3–4):

- `src/main/java/.../fraudguard/<module>` – one package per module: `banking`, `risk`, `warning`, `review`, `audit`, `identity`
- `src/main/resources/db/migration` – Flyway migrations in plain SQL (`V<n>__<description>.sql`)
- `src/test/java` – tests, mirroring the module packages
- Architecture and domain terms: `docs/architecture.md`, `docs/spec.md` (glossary)

## Conventions

- Names in code match the glossary in `docs/spec.md` (e.g. `HeldTransfer`, `RiskScenario`, `RiskThreshold`, `ReviewTimeLimit`, `AuditLog`).
- Link user stories, PRs and tests to features and requirements by ID (`F-0X`, `NFR-0X`).
- Modular monolith: a module uses another module only through its public service, never through its repositories or tables.
- Constructor injection; no field injection.
- Use Java records for DTOs and value objects; keep JPA entities inside their module and don't expose them in the REST API.
- REST errors use Spring `ProblemDetail`.
- The OpenAI client is used only inside the `warning` module, behind our own interface. Tests use a fake, never the real OpenAI API.
- Tests that touch the database run on real PostgreSQL (Testcontainers), not an in-memory database.
- Formatting: palantir-java-format via Spotless (D-002). Run `./mvnw spotless:apply` before committing; don't format by hand against it.

## Don't

These come from the regulatory and security requirements (`docs/spec.md`, section 5) – treat them as hard rules:

- Never send data that could identify the customer to an external AI service (NFR-03). Anonymise before the call and cover it with tests.
- Never let the system approve or reject a held transfer on its own – the final decision is always a person's, with a justification (F-08).
- Never update or delete audit log entries; the audit log is append-only (NFR-01).
- Never make transfer processing depend on the AI service being available; there must always be a fallback (NFR-04).
- Never execute a held transfer after its review time limit has passed (F-09).
- Never use real customer data in tests, fixtures or logs.
- Don't edit a database migration once it has been committed; add a new one.
- Don't put frontend code in this repo.
- Don't add dependencies without a justification in the PR.
- Never commit secrets to the repo.
