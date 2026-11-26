# Decision log: FraudGuard

> One decision = one short entry, **newest first**. Entries are not edited once written; changing a decision means a new entry with a "Supersedes" field, and the old one gets the note "Superseded by D-00X".
>
> Record a decision when we: choose or change a technology, database, service or library with wide impact; change an API contract or the data model; deliberately take on technical debt. The entry is written in the same PR as the change, and the PR also updates `architecture.md` if the structure changes.

<!-- Entry template – copy above the newest decision:

## D-00X: [Decision title] – YYYY-MM-DD
- Context: the problem we are solving and our constraints
- Decision: we choose ..., because ...
- Rejected: [option] – why not
- Consequences: what we gain, what we lose, what has to change
- Related: [task ID / PR], [D-00Y]
- Supersedes: [D-00Y] (if applicable)
-->

## D-003: Authentication between the web app and the API – 2026-11-26
- Context: the React SPA (separate repo) calls the Spring Boot API. Users are built-in test users with two roles, customer and fraud analyst (NFR-05). Every decision in the audit log must record who made it (NFR-01). The PoC runs locally and later on AWS. Credentials must stay out of reach of JavaScript in a banking context.
- Decision:
  - **Server-side session with an HttpOnly cookie**, managed by Spring Security. The SPA logs in with `POST /api/auth/login` (username and password; passwords stored as BCrypt hashes, test users from seed data) and logs out with `POST /api/auth/logout`, which invalidates the session.
  - Session cookie flags: `HttpOnly`, `SameSite=Lax`, and `Secure` outside local development.
  - **CSRF protection** on state-changing requests: Spring Security issues the token in an `XSRF-TOKEN` cookie, and the SPA sends it back in the `X-XSRF-TOKEN` header.
  - **SPA and API on the same origin**, so no CORS is needed. Locally the frontend dev server proxies `/api/*` to the API. On AWS one domain serves the SPA at `/` and the API at `/api/*`.
  - Authorization by role in the API (Spring Security). The SPA only hides what the user can't do.
- Rejected:
  - JWT in the `Authorization` header – a token readable by JavaScript is exposed to XSS, logout and revocation are hard, and refresh tokens add code.
  - OAuth2 / OIDC with an external identity provider (Keycloak, Cognito) – production-grade, but an extra service to run and configure for two roles and test users. It can be added later with the API acting as a backend for frontend (BFF), so the SPA keeps using the session cookie.
  - SPA and API on different origins with CORS and cross-site cookies – more configuration and weaker cookie protection, with no benefit for the PoC.
- Consequences:
  - The API has no CORS configuration. The frontend repo needs a dev-server proxy for `/api` (e.g. Vite's `server.proxy`) – the frontend build tool is not decided yet.
  - Sessions live in API memory, which is enough for a single local instance. With more than one API instance on AWS, sessions move to PostgreSQL (Spring Session JDBC) – to be decided with the AWS deployment.
  - The authenticated user from the session is the source of "who" in the audit log.
- Related: D-001

## D-002: Database migrations and code formatting – 2026-11-26
- Context: D-001 chose Java 25, Spring Boot, Maven and PostgreSQL, but left the migration tool and the linter / formatter open. The schema must support an append-only audit log protected at database level (NFR-01). Committed migrations must never change. Code style should be identical whether code is written in IntelliJ or by an AI agent, and checked in CI.
- Decision:
  - Migrations: **Flyway**. Migrations are plain SQL files in `src/main/resources/db/migration` (`V1__…sql`), run automatically when the API starts. Plain SQL lets us write `GRANT` / `REVOKE UPDATE, DELETE` and triggers for the audit table directly. Flyway's checksums stop the app if a committed migration has been changed.
  - Formatting: **Spotless** Maven plugin with **palantir-java-format** (4-space indent, 120-character lines, close to IntelliJ's default style). `./mvnw spotless:apply` formats the code, and `./mvnw spotless:check` runs in the build and fails it on unformatted code.
- Rejected:
  - Liquibase – more flexible (XML/YAML changelogs, rollbacks), but more configuration than the PoC needs, and plain SQL is clearer for the audit-log permissions.
  - google-java-format – same approach, but 2-space indent and 100-character lines fit Java/Spring code in IntelliJ less well.
  - Checkstyle – duplicates the formatter and needs its own rule set to maintain.
  - PMD, SpotBugs – too heavy for the start of the PoC.
- Consequences:
  - `./mvnw verify` fails on unformatted code, and CI enforces one style. Developers run `spotless:apply` or use the palantir-java-format IntelliJ plugin.
  - Migrations are append-only by convention and by Flyway checksums. A schema change is always a new `V…` file.
  - Static analysis (e.g. Error Prone) is not included yet. It can be added later as a separate decision, once its Java 25 support is confirmed.
- Related: D-001

## D-001: Technology stack for the PoC – 2026-11-26
- Context: FraudGuard is a standalone PoC: a web application for two roles (customer, fraud analyst) with a rules-based risk engine, an append-only audit log, a background job for the review time limit, built-in test users and one external integration (an AI service, with anonymisation and a fallback). It runs as a demo, not in production, but it should look credible to a bank.
- Decision:
  - Backend: Java 25 + Spring Boot (a version that supports Java 25), built with Maven, exposing a REST/JSON API. Spring Boot is standard in banking and covers REST, security, scheduling, persistence and migrations. Maven is declarative and the most common build tool in enterprise projects.
  - Database: PostgreSQL with schema migrations in the backend repo. It is relational, and the audit log can be protected at database level (no `UPDATE`/`DELETE`).
  - Frontend: React.js in a **separate repository**, talking to the backend only through the REST API.
  - AI: OpenAI API, behind our own interface in the backend, so the provider can be swapped and the template fallback is always available.
  - Running: locally with Docker Compose (backend, PostgreSQL, frontend) during development – one command for a demo.
  - Deployment: the finished project will be deployed on **AWS**. Which AWS services to use is a separate decision.
- Rejected:
  - Server-rendered UI (Thymeleaf + htmx) in the same app – simpler, but a React frontend in its own repo is preferred.
  - Gradle – faster and more flexible, but that advantage doesn't matter for a single-module PoC.
  - The IntelliJ module system (`.iml`) as the build – it works only inside the IDE, has no dependency management and can't build in CI or Docker.
  - Quarkus, plain Java + libraries – a smaller ecosystem or more wiring by hand, with no gain for this PoC.
  - Anthropic API / Amazon Bedrock as the AI provider – OpenAI preferred.
- Consequences:
  - Two repositories: the backend (this one) and the frontend. We need an API contract (e.g. OpenAPI) and CORS / authentication that works across origins – the authentication approach is a separate decision.
  - The `.iml`-based module is replaced by a Maven project; IntelliJ imports `pom.xml`.
  - OpenAI processes data outside the bank, so anonymisation before every call (NFR-03) is critical and must be covered by tests. The AI call has a timeout and a template fallback (NFR-04).
  - The AWS deployment needs its own decision: the services for the backend, the frontend and PostgreSQL, the region, infrastructure as code, and how the OpenAI API key is stored. Building everything in containers from the start (Docker Compose) keeps that move simple.
  - `architecture.md` (overview, containers, deployment), `README.md`, `CLAUDE.md` (commands, structure) and `docs/README.md` (stack, repositories) must be updated to match.
- Related: [task ID / PR]
