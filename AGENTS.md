# AGENTS.md

## Build and test
- Build:   `dotnet build eShop.Web.slnf`
- Test:    `dotnet test`
- Run:     `dotnet run --project src/eShop.AppHost/eShop.AppHost.csproj`
- Docker must be running before the AppHost starts.

## Conventions
- Domain logic belongs in `*.Domain`. Never reach into a DbContext
  from an API endpoint.
- Integration events are contracts. Changing one is a breaking change
  to every subscriber; call that out rather than editing quietly.
- Migrations are generated, never hand-edited.

## Do not touch
- Generated migration files under `*/Migrations/`.
- `eShop.ServiceDefaults` without flagging the cross-cutting impact.
