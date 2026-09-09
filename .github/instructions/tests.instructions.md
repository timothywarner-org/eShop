---
description: "Two test frameworks split by project suffix. Confirms which one applies before a test is written."
applyTo: "tests/**"
---

# Test instructions

Two frameworks here, split by project type. Confirm which one you are in
before writing a test.

- `*.UnitTests` build on `MSTest.Sdk`. MSTest attributes, NSubstitute for
  test doubles.
- `*.FunctionalTests` build on `Aspire.AppHost.Sdk` and use xUnit v3.
  These are the only tests allowed to start a container.

Regardless of framework:

- One assert concept per test; multiple assertions are fine when they
  describe the same fact.
- Name tests `Method_Scenario_ExpectedOutcome`.
- Unit tests never touch a database or a container. If a test needs
  either, it belongs in a `*.FunctionalTests` project instead.
- Do not write a test that passes by asserting on a mock you configured
  in the same test.
