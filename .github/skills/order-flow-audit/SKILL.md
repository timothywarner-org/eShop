---
name: order-flow-audit
description: Audit a change to the ordering service for order-of-side-effect
  hazards, transaction boundary violations, and integration event contract
  breaks. Use whenever code under src/Ordering.* is added, modified, or
  reviewed, or when asked how an order flows through the system.
---

# Order flow audit

## When this applies
Any change touching `src/Ordering.API`, `src/Ordering.Domain`, or
`src/Ordering.Infrastructure`, and any question about how an order
moves through the system.

## What to check

1. **Side effect ordering.** Does anything depend on a database-generated
   value being populated before it is read? Domain events dispatch inside
   the same transaction as `SaveChanges`, so this is the highest-frequency
   defect in this service.

2. **Transaction boundaries.** Does a handler perform work that must
   either commit with the aggregate or not at all? If it crosses the
   boundary, say what compensating action exists, or that none does.

3. **Integration event contracts.** Any change to an event shape is a
   breaking change to every subscriber. Enumerate the subscribers.

4. **Existing warnings.** Quote any `REVIEW` or `TODO` comment in the
   files under review. These are prior engineers telling you where the
   fragility is.

## How to report
Lead with the highest-severity finding. For each finding, give the file,
the mechanism, and what breaks in production. Do not soften a real hazard
into a style suggestion.
