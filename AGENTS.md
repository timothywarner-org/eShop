# AGENTS.md

## What this repository is

A teaching fork of `dotnet/eShop`: a .NET 10 e-commerce reference application,
service-per-bounded-context under `src/`, orchestrated by .NET Aspire, with
EF Core, MediatR, PostgreSQL, Redis, and RabbitMQ. About 20 projects in `src/`
and 9 test projects in `tests/`.

The application code is upstream material. The deliverable maintained here is
the context-engineering lesson layered on top of it. **Always default to
reading source rather than changing it.** Leave application code intact unless
the task explicitly calls for an application change.

## Build and test

Preconditions, in order. Check them before reporting a failure as a code defect.

1. **SDK.** `global.json` pins `10.0.302` with `rollForward: latestFeature`,
   which requires the `3xx` feature band or higher. A 10.0.1xx or 10.0.2xx SDK
   does not satisfy it and fails with a missing-SDK message rather than a
   compile error. Verify with `dotnet --version` at the repository root.
2. **Container runtime.** Required before the AppHost or any functional test
   starts. Docker Desktop is the default; Podman is supported.

| Task | Command |
| --- | --- |
| Build | `dotnet build eShop.Web.slnf` |
| Test | `dotnet test --solution eShop.Web.slnf` |
| Run | `aspire run` from the repository root |
| Browser journeys | `npm ci`, then `npx playwright install chromium`, then `npm run test:e2e` |

`eShop.Web.slnf` filters `eShop.slnx` to exclude the MAUI projects. Build and
test through the filter unless the task is specifically about `src/ClientApp`
or `src/HybridApp`.

**Verification status:** the build and test commands are the ones this
repository's own CI runs on every pull request, so they are correct for a
machine that satisfies the preconditions. They have not been executed on the
current machine. State which of these you actually ran before reporting a
result.

### Running one test

`global.json` selects the Microsoft.Testing.Platform runner, so `dotnet test`
takes MTP arguments, not VSTest arguments.

- `--project`, `--solution`, and `--test-modules` are mutually exclusive.
- Arguments for the test application go after a literal `--`.
- **Always scope to one project before filtering.** The two frameworks here
  reject each other's filter options, so a filter combined with `--solution`
  exits with code 5.

```powershell
dotnet test --project tests/Ordering.UnitTests/Ordering.UnitTests.csproj -- --filter "FullyQualifiedName~BuyerAggregateTest"
```

`--filter` is the MSTest option, valid for `*.UnitTests`. The xUnit v3
`*.FunctionalTests` projects reject it; check that project's `--help` for its
own filter options. Functional tests start containers.

`TreatWarningsAsErrors` is on, so a new warning fails the build. Build output
goes to `artifacts/`. Package versions belong in `Directory.Packages.props`,
never in a `.csproj`.

## Conventions

- Domain logic belongs in `*.Domain`. Never reach into a DbContext from an API
  endpoint.
- Integration events are contracts. Before changing their shape or meaning,
  enumerate the subscribers and identify breaking changes explicitly.
- Distinguish in-process MediatR domain events from RabbitMQ integration
  events. `TransactionBehavior` commits the transaction and then publishes the
  logged integration events; that order is the outbox guarantee.
- Migrations are generated, never hand-edited.

## Do not touch

- Generated migration files under `*/Migrations/`.
- `eShop.ServiceDefaults` without flagging the cross-cutting impact.

## Codebase navigation

- For an unfamiliar business flow, read `docs/repo-map.md` for entry points,
  then verify the relevant relationships in source. The map can become stale.
- When preserving a finding, keep its evidence and the reasoning rule that
  should guide the next investigation. State the rule's scope and why it
  matters; do not turn an untested hypothesis into a permanent instruction.

## Where new guidance belongs

Route knowledge by when it is needed, and keep this file short.

| Kind of knowledge | Destination |
| --- | --- |
| A navigation fact or a source path | `docs/repo-map.md` |
| A check that applies only to one subsystem | `.github/instructions/*.instructions.md` with an `applyTo` glob |
| A repeatable task with a fixed report format | `.github/prompts/*.prompt.md` |
| A multi-step procedure the agent should select on its own | `.github/skills/<name>/SKILL.md` |
| A persona plus a tool boundary | `.github/agents/*.agent.md` |
| A convention every request needs | This file, or `.github/copilot-instructions.md` |

## Lesson sources

- This repository is the canonical student and demo source for the lesson.
- Keep instructor scripts, cue sheets, private diagrams, and rehearsal material
  outside this repository. Never commit or push them to origin.
- Maintain the student journey in `README.md`, navigation in `docs/repo-map.md`,
  and the leave-behind in `docs/copilot-context-engineering.html`. Distributed
  copies are exports.
- Keep the handout's copyable examples aligned with the context files. Update
  both in the same change.
- **Never** include client identities, private session details, or internal
  metrics in any file, filename, commit message, or diagram in this repository.
  Public materials stay suitable for any student.
- Verify claims against source and current product docs. Distinguish a
  documented warning from a reproduced failure.
