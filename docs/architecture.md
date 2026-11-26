# Architecture: [project name]

Last updated: [YYYY-MM-DD]

> This file describes **how** the system is built. We use the **C4 model**: levels 1 and 2 are mandatory, level 3 only for complex containers, and we don't draw level 4 (code) – the code is the source of truth. Diagrams are written in Mermaid so they can be reviewed in PRs. Names on diagrams = names in the code and in the glossary (`spec.md`, section 7).

## 1. Overview

| Item | Value |
| --- | --- |
| Stack | [language, framework, database] |
| Hosting | [e.g. AWS / Vercel / VPS] |
| Environments | [dev – URL], [staging – URL], [prod – URL] |
| Repositories | [single repo / list] |

[2–3 sentences: the main idea of the solution, e.g. "a modular monolith with a REST API and an SPA; background jobs on a queue". The reasons for important choices are in `decisions.md`.]

## 2. System context (C4 – level 1)

Question: **who and what uses the system, and what does it integrate with?** The system is a single box.

```mermaid
C4Context
  title System context: [name]
  Person(user, "[User]", "[who they are]")
  Person(admin, "[Administrator]", "[who they are]")
  System(system, "[System name]", "[what it does in one sentence]")
  System_Ext(ext1, "[External system]", "[e.g. payment gateway]")
  System_Ext(ext2, "[External system]", "[e.g. email delivery]")
  Rel(user, system, "[Uses]", "HTTPS")
  Rel(admin, system, "[Manages]", "HTTPS")
  Rel(system, ext1, "[Requests payments]", "REST")
  Rel(system, ext2, "[Sends emails]", "API")
```

| Element | Responsibility |
| --- | --- |
| [External system] | [what we get from it / what we send to it] |

## 3. Containers (C4 – level 2)

Question: **which runnable parts make up the system, and how do they communicate?** Container = application, service, database, queue.

```mermaid
C4Container
  title Containers: [name]
  Person(user, "[User]")
  System_Boundary(sys, "[System name]") {
    Container(web, "[Web app]", "[e.g. React]", "[user interface]")
    Container(api, "[API]", "[e.g. Node.js]", "[business logic]")
    Container(worker, "[Worker]", "[e.g. Node.js]", "[background jobs]")
    ContainerDb(db, "[Database]", "[e.g. PostgreSQL]", "[domain data]")
    ContainerQueue(queue, "[Queue]", "[e.g. SQS]", "[asynchronous jobs]")
  }
  System_Ext(ext1, "[External system]")
  Rel(user, web, "Uses", "HTTPS")
  Rel(web, api, "Calls", "JSON/HTTPS")
  Rel(api, db, "Reads and writes", "SQL")
  Rel(api, queue, "Publishes jobs")
  Rel(worker, queue, "Consumes jobs")
  Rel(api, ext1, "[Requests payments]", "REST")
```

| Container | Technology | Responsibility | Code |
| --- | --- | --- | --- |
| [API] | [Node.js] | [business logic, authorization] | `[src/api]` |
| [Database] | [PostgreSQL] | [domain data] | `[migrations/]` |

## 4. Components (C4 – level 3, optional)

Draw only for a container whose structure is not obvious from the code. One diagram per container.

```mermaid
C4Component
  title Components: [API]
  Container(web, "[Web app]", "[React]")
  ContainerDb(db, "[Database]", "[PostgreSQL]")
  Container_Boundary(api, "[API]") {
    Component(auth, "[Auth module]", "[e.g. module]", "[sign-in, tokens]")
    Component(orders, "[Orders module]", "[e.g. module]", "[placing orders, order statuses]")
    Component(repo, "[Data layer]", "[e.g. ORM]", "[database access]")
  }
  Rel(web, auth, "Signs in", "JSON/HTTPS")
  Rel(web, orders, "Places orders", "JSON/HTTPS")
  Rel(orders, repo, "Uses")
  Rel(repo, db, "SQL")
```

## 5. Key flows

A sequence diagram only for the most important or non-obvious scenarios (usually 1–3). Each one points to a feature in `spec.md`.

### 5.1 [Scenario name] (F-0X)

```mermaid
sequenceDiagram
  actor U as [User]
  participant W as [Web app]
  participant A as [API]
  participant D as [Database]
  participant E as [External system]
  U->>W: [action]
  W->>A: [POST /resource]
  A->>D: [write]
  A->>E: [call]
  E-->>A: [response]
  A-->>W: [201 Created]
  W-->>U: [confirmation]
```

Error cases: [what happens when the external system does not respond / the data is invalid].

## 6. Deployment

| Container | Where it runs | How we deploy |
| --- | --- | --- |
| [Web app] | [e.g. CDN / Vercel] | [automatically from `main`] |
| [API] | [e.g. container on ECS] | [CI pipeline on tag] |
| [Database] | [e.g. RDS] | [migrations in the pipeline] |

Secrets and configuration: [where they are kept, e.g. a secrets manager; never in the repo].

## 7. Cross-cutting concepts

Briefly, only what affects many places in the code:

- **Authentication and authorization:** [e.g. JWT, roles: customer, admin]
- **Error handling and logging:** [e.g. log format, where logs go]
- **Monitoring:** [e.g. metrics, alerts]
- **Personal data:** [what we store, how we protect it, how we delete it]
