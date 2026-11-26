# Specification: FraudGuard

Version 0.1 · 2026-11-26 · Owner: Michał Bąk

> This file describes **what** we are building and **why**. It changes rarely; concrete work lives in the tracker as user stories (see `docs/README.md`, section 2.2). Any change of scope = an update to this file in a PR.
>
> Source: [REQUIREMENTS.md](../REQUIREMENTS.md). References like `(R-5)` point to the numbered requirements there.

## 1. Goal

The bank loses money to fraud in which customers send transfers themselves while being manipulated, for example by a phone call from someone pretending to be a bank employee. FraudGuard checks every transfer for fraud risk before it is sent, warns the customer and asks short control questions when a transfer looks suspicious, and passes doubtful cases to a bank employee for review – without slowing down ordinary transfers.

**This version is a standalone proof of concept (PoC).** It is not integrated with any bank system. FraudGuard simulates accounts, transfers, transfer history and users with test data, and its only external dependency is an AI service. The PoC demonstrates the flow and the rules. The success measures below apply to a future production version and cannot be measured on test data.

Success measures (thresholds to be agreed with the bank):

- lower losses from transfer fraud – [target, e.g. −X% year over year]
- fewer complaints related to transfer fraud – [target]
- low false-alarm rate (ordinary transfers held without reason) – [target, e.g. < X% of all transfers]
- short review time for held transfers – [target, e.g. median < X min]

## 2. Users

| Role | Who they are | What they need |
| --- | --- | --- |
| Customer | bank customer sending a transfer | send ordinary transfers without friction; be warned clearly when they may be being scammed; stay in control of a held transfer |
| Fraud analyst | bank employee on the fraud team | a prioritised queue of held transfers with all context needed to decide quickly and justify the decision; define risk scenarios and the holding threshold without involving the vendor |

Auditors and the financial supervisor (KNF) are not users of the system. They receive explanations of decisions from the bank, and the audit log makes those explanations possible (NFR-01, NFR-02).

## 3. Scope

Features at a level a non-technical person understands. Details and acceptance criteria are in the user stories.

| ID | Feature | For whom | Priority | Source |
| --- | --- | --- | --- | --- |
| F-01 | Risk assessment of every transfer before it is sent; low-risk transfers are executed immediately with no extra steps | Customer | Must | R-1, R-2 |
| F-02 | Plain-language warning for a suspicious transfer: why it is risky and what type of scam it resembles, marked as prepared with the help of AI | Customer | Must | R-3, R-8 |
| F-03 | Control questions (e.g. "Is someone on the phone with you right now?"); with no concerns the customer can knowingly confirm, otherwise the transfer goes to review | Customer | Must | R-4, R-5 |
| F-04 | Cancelling a held transfer at any time, with funds released | Customer | Must | R-6 |
| F-05 | Informing the customer of the decision on a held transfer and of the expected review time | Customer | Must | R-7 |
| F-06 | Review queue of held transfers, ordered from the riskiest | Fraud analyst | Must | R-9 |
| F-07 | Review details: why the system stopped the transfer, the customer's answers, and how the customer normally uses the account | Fraud analyst | Must | R-10 |
| F-08 | Approve / reject decision made by a person, always with a short justification | Fraud analyst | Must | R-11 |
| F-09 | Review time limit: a transfer not reviewed in time is not executed | Fraud analyst, Customer | Must | R-12 |
| F-10 | Configuration of risk scenarios and of the risk threshold above which a transfer is held, by the bank itself | Fraud analyst | Must | R-13 |

Priorities: **Must** (we don't launch without it), **Should** (important, but can wait), **Could** (if time allows). All requirements in `REQUIREMENTS.md` are currently treated as Must; priorities to be confirmed with the bank.

## 4. Out of scope

What we deliberately do **not** do (at least in this version):

- integration with bank systems (online banking, transfer system, account history, identity and access, notifications) – the PoC simulates them internally
- real customer data – the PoC uses test data only
- automatic approval or rejection of a held transfer by the system – the final decision is always made by a person (R-11)
- sending any customer-identifying data to external AI services (R-15)
- [to confirm with the bank] fraud without the customer's involvement, e.g. account takeover or stolen credentials
- [to confirm with the bank] channels and transfer types not listed in the requirements (e.g. card payments, BLIK)

## 5. Non-functional requirements

Every requirement must be measurable. "The system should be fast" is not a requirement.

| ID | Category | Requirement and threshold | How we verify |
| --- | --- | --- | --- |
| NFR-01 | Audit | Every decision (system or employee) is recorded permanently and cannot be changed or deleted: who, when, on what basis (risk score, triggered scenarios, customer answers, justification). Retention period: [TBD with the bank] (R-14) | tests on the audit log; check that no update/delete path exists; audit before release |
| NFR-02 | Explainability | For every held or released transfer the bank can produce an explanation understandable to the customer, an auditor and KNF (R-14) | sample of decisions reviewed by the fraud team before release |
| NFR-03 | Privacy | 0 fields that could identify the customer (name, account number, ID numbers, address, phone, e-mail, etc.) in requests to external AI services (R-15) | automated tests on outgoing payloads; review of the anonymisation code |
| NFR-04 | Resilience | When the AI service is unavailable or times out, transfers are still processed: risk assessment works and a simplified (template) warning is shown. AI call timeout: [TBD] (R-16) | tests with the AI service disabled / slowed down |
| NFR-05 | Security / access | Customers access only their own accounts and transfers; employees see only the data needed for a review (R-17) | authorisation tests per role; security scan in CI |
| NFR-06 | Performance | Risk assessment adds at most [TBD, e.g. p95 < 200 ms] to a low-risk transfer (R-2) | load test |
| NFR-07 | Availability | [TBD, e.g. 99.9% per month] – the check is on the path of every transfer | monitoring |

## 6. Constraints and assumptions

- Constraints: Polish banking regulation and KNF supervision; GDPR; decisions must remain explainable; the bank configures risk rules without the vendor; [technology stack – to be decided, see `docs/decisions.md`]
- Assumptions: the PoC is standalone. Accounts, balances, transfer execution, transfer history (for the customer profile), users and roles, and customer messages are simulated inside FraudGuard with test data. The regulatory rules (audit log, anonymisation before AI, AI fallback, access control) are implemented as they would be in production.
- External dependencies: only the external AI service used to prepare warnings (optional at runtime, see NFR-04). [Which AI service – TBD]

## 7. Glossary

We use the same names in code, on diagrams and in user stories.

| Term | Meaning |
| --- | --- |
| Customer | a bank customer who owns the account and sends the transfer |
| Transfer | a payment order created by the customer, checked by FraudGuard before it is sent |
| Risk assessment | the check of a transfer before execution that produces a risk score and the list of triggered risk scenarios |
| Risk score | a number describing how likely the transfer is to be fraud |
| Risk scenario | a situation the bank considers risky (e.g. new recipient + unusually high amount), defined and maintained by the bank |
| Risk threshold | the risk score above which a transfer is held; set by the bank |
| Held transfer | a transfer stopped before execution, waiting for the customer's confirmation or a review |
| Scam type | the category of fraud a transfer resembles (e.g. fake bank employee, investment scam) |
| Warning | the plain-language message shown to the customer for a suspicious transfer, prepared with the help of AI |
| Simplified warning | a template warning used when the AI service is unavailable |
| Control questions | short questions the customer answers for a held transfer (e.g. whether someone is on the phone with them) |
| Review | the check of a held transfer by a fraud analyst |
| Review queue | the list of held transfers waiting for review, ordered from the riskiest |
| Review time limit | the time within which a held transfer must be reviewed; after it the transfer is not executed |
| Decision | the outcome for a transfer – by the system (release, hold) or by a fraud analyst (approve, reject, with justification) |
| Audit log | the permanent, unchangeable record of all decisions: who, when, on what basis |
| Customer profile | how the customer normally uses their account (typical amounts, recipients, times), shown during review |
| Identifying data | any data that could identify the customer; never sent to external AI services |
| Fraud analyst | a bank employee who reviews held transfers and manages risk scenarios and the risk threshold |
