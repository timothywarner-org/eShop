---
applyTo: "src/Ordering.API/**,src/Ordering.Domain/**,src/Ordering.Infrastructure/**"
---

# Ordering service instructions

This is a Domain-Driven Design service. Aggregates own their invariants;
command handlers orchestrate but do not contain business rules.

Domain events dispatch through MediatR inside `OrderingContext`, and
they run in the same transaction as `SaveChanges`. That has consequences:

- Anything that depends on a database-generated Id being populated is
  fragile. Flag it every time you see it.
- If a handler yields before `SaveChanges` completes, ordering guarantees
  you may be assuming do not hold.

When asked to explain or modify an ordering flow, always report:
1. The command, the aggregate, and the handlers involved.
2. Every point where correctness depends on execution order.
3. Any existing `REVIEW` or `TODO` comment in the files you touched.
