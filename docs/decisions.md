# Decision log: [project name]

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

## D-001: [Choice of technology stack] – [YYYY-MM-DD]
- Context: [e.g. small team, need for a quick start, hosting in the EU]
- Decision: [e.g. Node.js + PostgreSQL + React], because [the team knows these technologies, good hosting support]
- Rejected: [e.g. Python + Django] – [less team experience]
- Consequences: [one language on frontend and backend; typing needs care – we use TypeScript]
- Related: [PROJ-1]
