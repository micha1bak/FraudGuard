# FraudGuard

A system that checks every bank transfer for fraud risk before it is sent, warns customers who may be being manipulated by a scammer (for example a fake "bank employee" on the phone), and routes suspicious transfers to the bank's fraud team for review by a person.

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

## Requirements

- Java (IntelliJ IDEA module; JDK version not chosen yet)
- [Build tool, database, other services – to be decided, see [Decisions](docs/decisions.md)]

## Getting started

> The technology stack has not been chosen yet. This section will be filled in once the build setup exists.

```bash
[cp .env.example .env]
[build command]
[run command]
```

The app runs at: [URL – TBD]

## Tests and quality

```bash
[test command]     # tests
[lint command]     # linter and formatter
```

## Configuration

> To be filled in as configuration is introduced. Expected areas, based on the requirements:

| Variable | Description | Example |
| --- | --- | --- |
| [DATABASE_URL] | database connection (transfers, decisions, audit log) | [TBD] |
| [AI_SERVICE_URL] | external AI service used for warnings; the system must work without it | [TBD] |
| [REVIEW_TIMEOUT] | time after which an unreviewed held transfer is not executed | [TBD] |

## Documentation

- [Requirements](REQUIREMENTS.md) – business, customer, fraud team and regulatory requirements
- [Working rules](docs/README.md) – PRs, branches, Definition of Done, AI agents
- [Specification](docs/spec.md) – goal, scope, non-functional requirements
- [Architecture](docs/architecture.md) – C4 model, flows, deployment
- [Decisions](docs/decisions.md) – decision log
- [Changelog](CHANGELOG.md)
