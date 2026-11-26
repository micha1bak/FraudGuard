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
