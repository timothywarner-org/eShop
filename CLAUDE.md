# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

Tim Warner's teaching fork of [dotnet/eShop](https://github.com/dotnet/eShop), published as `timothywarner-org/eShop`. The application code is upstream .NET reference material. The **deliverable** is the context-engineering lesson layered on top of it.

**Default to reading source, not changing it.** `AGENTS.md` states the rule: leave application code intact unless the task explicitly calls for an application change. Most work here is investigation, documentation, or edits to the context files below.

## Read these before acting

This repository already carries an agent context layer. Follow it; do not restate it here.

| File | What it governs |
| --- | --- |
| `AGENTS.md` | Build/test commands, edit boundaries, how to preserve a finding |
| `.github/copilot-instructions.md` | Stack facts, evidence requirements, uncertainty reporting |
| `.github/instructions/ordering.instructions.md` | Applies to `src/Ordering.*`: aggregates, ID availability, transaction boundaries |
| `.github/instructions/tests.instructions.md` | Applies to `tests/**`: which framework, what belongs in unit vs functional |
| `.github/prompts/trace-flow.prompt.md` | The six-section flow-tracing report format |
| `.github/skills/order-flow-audit/SKILL.md` | Review procedure for Ordering changes |
| `.github/agents/flow-investigator.agent.md` | Read-only investigation persona; granted only the `search` and `read` tool sets |
| `docs/repo-map.md` | Navigation: entry points and verified relationships, with an evidence baseline commit |
| `docs/try-these-prompts.md` | The five classroom prompts and what each one is meant to expose |

The map is ordinary documentation and can go stale. Use it to pick a starting point, then confirm against source.

## Build, run, test

Prerequisites: a .NET SDK satisfying `global.json`, the [Aspire CLI](https://aspire.dev/get-started/install-cli/), and a running container runtime.

| Task | Command |
| --- | --- |
| Build the web solution | `dotnet build eShop.Web.slnf` |
| Test the web solution | `dotnet test --solution eShop.Web.slnf` |
| Run the app | `aspire run` (repo root; `aspire.config.json` selects the AppHost) |
| Run in background | `aspire start`, then `aspire ps`, then `aspire stop` |
| Browser journeys | `npm ci`, `npx playwright install chromium`, `npm run test:e2e` |

`eShop.Web.slnf` is a filter over `eShop.slnx` that omits the MAUI projects (`src/ClientApp`, `src/HybridApp`, `tests/ClientApp.UnitTests`). Those build in a separate workflow that installs MAUI workloads. Use the filter unless the task is specifically mobile.

### SDK pin

`global.json` requests `10.0.302` with `rollForward: latestFeature`, which needs the `3xx` feature band or higher. A 10.0.1xx or 10.0.2xx SDK does **not** satisfy it, and the failure reads as a missing-SDK error rather than a code error. Run `dotnet --version` at the repo root before diagnosing a build break.

### Running a single test

`global.json` sets `"test": { "runner": "Microsoft.Testing.Platform" }`, so `dotnet test` uses MTP argument syntax, not VSTest syntax.

- `--project`, `--solution`, and `--test-modules` are **mutually exclusive**.
- Arguments intended for the test application go after a literal `--`.
- The two test frameworks reject each other's filter options, so **never pass a filter alongside `--solution`**. That returns exit code 5. Scope to one project first.

```powershell
# MSTest project (*.UnitTests): --filter is the correct option
dotnet test --project tests/Ordering.UnitTests/Ordering.UnitTests.csproj -- --filter "FullyQualifiedName~BuyerAggregateTest"
```

For an xUnit v3 project (`*.FunctionalTests`), `--filter` is not valid; `--filter-trait` is documented, and the project's own `--help` lists the rest. Functional tests start containers through the Aspire AppHost, so the container runtime must be running.

### Build settings that bite

- `TreatWarningsAsErrors` is `true` in `Directory.Build.props`. A new warning fails the build.
- `UseArtifactsOutput` is `true`. Build output goes to `artifacts/`, not per-project `bin/`.
- Central package management is on. Add or change versions in `Directory.Packages.props`, never in a `.csproj`.

## Architecture

### Service per bounded context

Each service under `src/` owns its own data store. `src/eShop.AppHost/Program.cs` is the single place that declares what exists and what connects to what: PostgreSQL (`catalogdb`, `identitydb`, `orderingdb`, `webhooksdb`, on the `ankane/pgvector` image), Redis for Basket, RabbitMQ as the event bus, and a YARP `mobile-bff` reverse proxy. `WebApp` reaches Basket over gRPC and reaches Catalog and Ordering over HTTP.

Read the AppHost for wiring, then read the consuming service's `Extensions/Extensions.cs` for what it actually does with that resource. A `WithReference` establishes a connection, not a behavior.

Startup order encodes a real dependency: `order-processor` waits for `ordering-api` because Ordering.API applies the EF migrations.

### Two event mechanisms, easily conflated

This is the distinction that requires reading several files together, and it is the heart of the lesson.

| | Domain events | Integration events |
| --- | --- | --- |
| Transport | MediatR, in process | RabbitMQ, via `src/EventBusRabbitMQ` |
| Scope | Inside one Ordering transaction | Across service boundaries |
| Queued by | The aggregate, during its own methods | Handlers, into the log in `src/IntegrationEventLogEF` |
| Dispatched by | `OrderingContext.SaveEntitiesAsync`, which publishes queued events **before** calling `SaveChangesAsync` (see `src/Ordering.Infrastructure/MediatorExtension.cs`) | `TransactionBehavior`, **after** the transaction commits |

`src/Ordering.API/Application/Behaviors/TransactionBehavior.cs` wraps each command: begin transaction, run the handler, commit, then `PublishEventsThroughEventBusAsync`. That ordering is the outbox guarantee. Preserve it.

### Ordering is DDD; Catalog and Basket are not

`src/Ordering.Domain` holds aggregates that own their invariants. Command handlers in `src/Ordering.API/Application/Commands` orchestrate and must not carry business rules. Catalog and Basket are deliberately simpler data-driven services; do not impose the Ordering patterns on them.

### The worked example, and how to treat it

The lesson's investigation follows order placement: buyer and payment verification, the numeric `Buyer.Id` and `Payment.Id` consumed by `UpdateOrderWhenBuyerAndPaymentMethodVerifiedDomainEventHandler`, the separate `BuyerIdentityGuid` carried by `OrderStatusChangedToSubmittedIntegrationEvent`, and `UseHiLo` key generation configured in the Ordering entity configurations.

There is a `REVIEW` comment in `ValidateOrAddBuyerAggregateWhenOrderStartedDomainEventHandler.cs`. **Treat it as a claim to investigate, not a reproduced failure.** Quote it, check it against the current implementation, and label anything untested as predicted. Do not fix it as a drive-by; the lesson depends on the code staying as it is.

Three `REVIEW` comments exist in `src/` in total. The other two are in `Catalog.API/Extensions/Extensions.cs` and `EventBusRabbitMQ/RabbitMQEventBus.cs`.

## Editing boundaries

- **Do not hand-edit** anything under `*/Migrations/` in Catalog.API, Identity.API, Ordering.Infrastructure, or Webhooks.API. Migrations are generated.
- **Flag cross-cutting impact** before changing `src/eShop.ServiceDefaults`. Every service consumes it.
- **Integration events are contracts.** Before changing an event's shape or meaning, enumerate its subscribers and state which would break and why.
- Domain logic belongs in `*.Domain`. Do not reach into a DbContext from an API endpoint.

## Lesson content rules

- Keep the student journey in `README.md`, navigation in `docs/repo-map.md`, and the leave-behind handout in `docs/copilot-context-engineering.html`. Distributed copies are exports, not sources.
- The handout's copyable examples must stay aligned with the actual context files. Update both in the same change.
- When a map entry's underlying source changes, recheck the entry and record the new evidence baseline commit in the same change.
- Keep public materials audience-neutral: no client identities, private session details, or internal metrics.
- Instructor scripts, cue sheets, and rehearsal material stay out of this repository entirely.
