---
applyTo: "tests/**"
---

# Test instructions

- xUnit. One assert concept per test; multiple assertions are fine when
  they describe the same fact.
- Name tests `Method_Scenario_ExpectedOutcome`.
- Unit tests never touch a database or a container. If a test needs
  either, it belongs in a `*.FunctionalTests` project instead.
- Do not write a test that passes by asserting on a mock you configured
  in the same test.
