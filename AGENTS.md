# AGENTS.md

## Build and test
- Build:   `dotnet build eShop.Web.slnf`
- Test:    `dotnet test --solution eShop.Web.slnf`
- Run:     `dotnet run --project src/eShop.AppHost/eShop.AppHost.csproj`
- Docker must be running before the AppHost starts.

## Conventions
- Domain logic belongs in `*.Domain`. Never reach into a DbContext
  from an API endpoint.
- Integration events are contracts. Check subscriber compatibility before
  changing their shape or meaning; identify breaking changes explicitly.
- Migrations are generated, never hand-edited.

## Do not touch
- Generated migration files under `*/Migrations/`.
- `eShop.ServiceDefaults` without flagging the cross-cutting impact.

## Lesson sources
- This repository is the canonical student and demo source for the lesson.
- Keep instructor scripts, cue sheets, private diagrams, and rehearsal
  material outside this repository. Do not commit or push them to origin.
- Maintain the student journey in `README.md` and the leave-behind in
  `docs/copilot-context-engineering.html`. Distributed copies are exports.
- Keep the handout's six copyable examples aligned with the context files.
- Keep public materials audience-neutral and free of client identities,
  private session details, and internal metrics.
- Verify claims against source and current product docs. Distinguish a
  documented warning from a reproduced failure; leave application code intact
  unless the task explicitly calls for an application change.
