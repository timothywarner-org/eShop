# Repository instructions

Stack: .NET 9, ASP.NET Core, .NET Aspire orchestration, EF Core, MediatR.
Architecture: service-per-bounded-context under `src/`, integration via
an event bus. Ordering is a DDD service; Catalog and Basket are simpler
data-driven services.

When explaining any cross-service flow:
- Name every service the flow touches, in order.
- Surface any `REVIEW`, `TODO`, `HACK`, or `FIXME` comment you find in
  the files involved. Quote it. Do not summarize it away.
- Say explicitly when correctness depends on the order of side effects.

When you cannot find something, say so and name what you searched.
Do not infer the existence of a class or method you have not seen.
