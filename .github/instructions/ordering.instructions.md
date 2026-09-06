---
applyTo: "src/Ordering.API/**,src/Ordering.Domain/**,src/Ordering.Infrastructure/**"
---

# Ordering service instructions

This is a Domain-Driven Design service. Aggregates own their invariants;
command handlers orchestrate but do not contain business rules.

`OrderingContext.SaveEntitiesAsync` dispatches domain events through
MediatR before calling `SaveChangesAsync`. Trace the surrounding transaction
and key-generation configuration before claiming an ordering guarantee.

- Check when numeric buyer and payment IDs become available. The mappings
  use HiLo; do not assume all generated keys arrive only at save time.
- Distinguish `Buyer.Id` from `Buyer.IdentityGuid` in integration events.
- Treat existing warning comments as claims to investigate against the
  current implementation, not proof that a runtime failure was reproduced.

When asked to explain or modify an ordering flow, always report:
1. The command, the aggregate, and the handlers involved.
2. Every point where correctness depends on execution order.
3. Any existing `REVIEW` or `TODO` comment in the files you touched.
