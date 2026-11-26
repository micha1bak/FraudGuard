# FraudGuard

A system that checks every bank transfer for fraud risk before it is sent, warns customers who may be being manipulated by a scammer (for example a fake "bank employee" on the phone), and routes suspicious transfers to the bank's fraud team for review by a person.

**Status:** standalone proof of concept (PoC). It is not integrated with any bank system: accounts, transfers and users are simulated with test data, and the only external service is the OpenAI API.

This repository holds the **backend** (Java 25, Spring Boot, PostgreSQL). The React frontend lives in a separate repository: [frontend repo – link TBD].

## What it does

**For customers**

- Every transfer is risk-scored before it is sent; low-risk transfers go through immediately, with no extra steps.
- A suspicious transfer is held, and the customer sees a plain-language warning: why it looks risky and what type of scam it resembles. The warning is marked as prepared with the help of AI.
- The customer answers a few short control questions (e.g. "Is someone on the phone with you right now?"). If the answers raise no concerns, they can knowingly confirm the transfer; otherwise it goes to a bank employee for review.
- The customer can cancel a held transfer at any time (funds are released) and is told the bank's decision and how long the review will take.

**For the fraud team**

- A queue of held transfers, ordered from the riskiest.
- For each transfer: why it was stopped, the customer's answers, and how the customer normally uses their account.
- Approve / reject is always a human decision with a short justification. A transfer not reviewed within the set time is not executed.
- The bank configures risk scenarios and the holding threshold itself, without the vendor.

**Key guarantees**

- Every decision (system or employee) is permanently recorded: who, when, and on what basis – explainable to the customer, auditors and the KNF.
- No customer-identifying data is sent to external AI services.
- An AI service outage does not stop transfers: risk assessment and warnings keep working in a simplified form.
- Customers see only their own accounts and transfers; employees see only what a review needs.

Full requirements: [REQUIREMENTS.md](REQUIREMENTS.md).

## Tech stack

| Part | Technology |
| --- | --- |
| Backend (this repo) | Java 25, Spring Boot, Maven |
| Database | PostgreSQL |
| Frontend (separate repo) | React.js |
| AI | OpenAI API (optional at runtime – a template warning is used when it is unavailable) |
| Local run | Docker Compose |
| Target deployment | AWS (services to be decided) |

Reasons and rejected options: [D-001](docs/decisions.md). How the parts fit together: [Architecture](docs/architecture.md).

## Requirements

- JDK 25
- Maven (or the Maven Wrapper `./mvnw`, once added to the repo)
- Docker with Docker Compose
- an OpenAI API key (optional – without it the app uses template warnings)

## Getting started

> **Planned setup.** The Maven project, `docker-compose.yml` and `.env.example` don't exist in the repo yet. The commands below show the intended workflow and will be confirmed once the code is in place.

```bash
cp .env.example .env              # fill in the OpenAI API key and database password
docker compose up -d db           # start PostgreSQL
./mvnw spring-boot:run            # start the API (Flyway database migrations run on start)
```

Or run everything in containers:

```bash
docker compose up
```

The API runs at: http://localhost:8080 (Spring Boot default; to be confirmed)

The frontend is started from its own repository. Its dev server proxies `/api/*` to the API, so both share one origin (D-003).

## Tests and quality

```bash
./mvnw test       # unit tests
./mvnw verify     # all tests, including integration tests on a real PostgreSQL (Testcontainers)
./mvnw spotless:apply   # format the code (palantir-java-format)
./mvnw spotless:check   # check formatting – also part of ./mvnw verify
```

## Configuration

Configuration is passed as environment variables (locally in an untracked `.env` file). Never commit secrets.

> Variable names are planned and will be confirmed once the code is in place.

| Variable | Description | Example |
| --- | --- | --- |
| `SPRING_DATASOURCE_URL` | PostgreSQL connection | `jdbc:postgresql://localhost:5432/fraudguard` |
| `SPRING_DATASOURCE_USERNAME` | database user | `fraudguard` |
| `SPRING_DATASOURCE_PASSWORD` | database password | – (secret) |
| `OPENAI_API_KEY` | OpenAI API key; when empty or OpenAI fails, template warnings are used | – (secret) |
| `OPENAI_TIMEOUT` | maximum wait for OpenAI before falling back to the template warning | `5s` |
| `FRAUDGUARD_REVIEW_TIME_LIMIT` | time after which a held transfer that has not been reviewed is not executed | `PT30M` |

## Documentation

- [Requirements](REQUIREMENTS.md) – business, customer, fraud team and regulatory requirements
- [Working rules](docs/README.md) – PRs, branches, Definition of Done, AI agents
- [Specification](docs/spec.md) – goal, scope, non-functional requirements
- [Architecture](docs/architecture.md) – C4 model, flows, deployment
- [Decisions](docs/decisions.md) – decision log
- [Changelog](CHANGELOG.md)
