# Specification: [project name]

Version [0.1] · [YYYY-MM-DD] · Owner: [person]

> This file describes **what** we are building and **why**. It changes rarely; concrete work lives in the tracker as user stories (see `docs/README.md`, section 2.2). Any change of scope = an update to this file in a PR.

## 1. Goal

[2–3 sentences: what problem we solve, for whom, and how we will know the project succeeded.]

Success measures:

- [e.g. a user places an order in under 2 minutes]
- [e.g. 100 active users within 3 months of launch]

## 2. Users

| Role | Who they are | What they need |
| --- | --- | --- |
| [Customer] | [description] | [main need] |
| [Administrator] | [description] | [main need] |

## 3. Scope

Features at a level a non-technical person understands. Details and acceptance criteria are in the user stories.

| ID | Feature | For whom | Priority |
| --- | --- | --- | --- |
| F-01 | [e.g. Sign-up and sign-in] | [Customer] | Must |
| F-02 | [e.g. Placing an order] | [Customer] | Must |
| F-03 | [e.g. Sales report] | [Administrator] | Should |

Priorities: **Must** (we don't launch without it), **Should** (important, but can wait), **Could** (if time allows).

## 4. Out of scope

What we deliberately do **not** do (at least in this version):

- [e.g. a mobile app]
- [e.g. payments in currencies other than EUR]

## 5. Non-functional requirements

Every requirement must be measurable. "The system should be fast" is not a requirement.

| ID | Category | Requirement and threshold | How we verify |
| --- | --- | --- | --- |
| NFR-01 | Performance | [API p95 response < 300 ms at 50 RPS] | load test |
| NFR-02 | Availability | [99.5% per month] | monitoring |
| NFR-03 | Security | [no critical vulnerabilities; passwords hashed] | scan in CI |
| NFR-04 | Usability | [WCAG 2.2 AA] | audit before release |
| NFR-05 | Data | [daily backup, restore < 4 h] | restore test |

## 6. Constraints and assumptions

- Constraints: [e.g. budget, deadline, required technology, GDPR, hosting in the EU]
- Assumptions: [e.g. users have a Google account; the payment provider offers an API]
- External dependencies: [e.g. provider X's API]

## 7. Glossary

We use the same names in code, on diagrams and in user stories.

| Term | Meaning |
| --- | --- |
| [Order] | [definition] |
| [Customer] | [definition] |
