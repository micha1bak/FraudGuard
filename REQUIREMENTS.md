# FraudGuard – Requirements

## Business goal

The bank wants to reduce losses from fraud in which customers send transfers themselves while being manipulated, for example by a phone call from someone pretending to be a bank employee. At the same time, ordinary transfers must not be slowed down or made harder to complete.

## Customer-facing requirements

1. Every transfer is checked for fraud risk before it is sent.
2. Low-risk transfers are executed immediately, with no extra steps for the customer.
3. For a suspicious transfer, the customer sees a warning in plain language explaining why the bank considers it risky and what type of scam it resembles.
4. The customer answers a few short control questions, for example whether someone is talking to them on the phone right now or whether someone told them to make this transfer.
5. If the answers raise no concerns, the customer can knowingly confirm the transfer. If they do, the transfer is passed on for review by a bank employee.
6. The customer can cancel a held transfer at any time, and the funds become available again.
7. The customer is informed of the bank's decision on a held transfer and of how long the review will take.
8. The customer is told that the warning was prepared with the help of artificial intelligence.

## Fraud team requirements

9. The employee sees a list of held transfers, ordered from the riskiest.
10. For each transfer, the employee can see why the system stopped it, how the customer answered the control questions, and how the customer normally uses their account.
11. The decision to approve or reject a transfer is always made by a person, not by the system. Every decision must include a short justification.
12. A transfer that has not been reviewed within the set time is not executed.
13. The bank can define on its own which situations it considers risky and above which risk level a transfer is held, without involving the vendor.

## Regulatory and security requirements

14. Every decision, whether made by the system or by an employee, is permanently recorded: who made it, when, and on what basis. The bank must be able to explain it to the customer, to auditors and to the financial supervisor (KNF).
15. No data that could identify the customer is sent to external AI services.
16. An outage of the AI service must not stop transfers. Risk assessment and warnings must keep working without it, even if in a simplified form.
17. Customers can access only their own accounts and transfers, and employees only the information needed to carry out a review.

## How the bank will measure success

Lower losses from transfer fraud, fewer related complaints, a low rate of false alarms (ordinary transfers held without reason), and a short review time for held transfers.
