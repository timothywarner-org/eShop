# Security policy

## What this repository is

This is a **fork of [dotnet/eShop](https://github.com/dotnet/eShop)** maintained as a teaching
artifact. It adds documentation and GitHub Copilot instruction files. It does not modify the
upstream application code.

Treat it accordingly: it is a reference sample used in training, not production software, and
it inherits every characteristic of the upstream sample including its default credentials,
development certificates, and local-only configuration. Do not deploy it to a public
environment.

## Reporting a vulnerability

Choose the destination by where the defect is, not by where you found it.

| The defect is in | Report it to |
| --- | --- |
| Upstream eShop application code, that is anything under `src/`, `tests/`, or `e2e/` | The upstream repository, at [github.com/dotnet/eShop/security](https://github.com/dotnet/eShop/security) |
| Files added by this fork, listed below | This repository, using **Report a vulnerability** on the [Security tab](https://github.com/timothywarner-org/eShop/security) |

Private vulnerability reporting is enabled here, so use the Security tab rather than opening a
public issue for anything sensitive.

Files added by this fork:

- `.github/copilot-instructions.md`
- `.github/instructions/ordering.instructions.md`
- `.github/instructions/tests.instructions.md`
- `.github/prompts/trace-flow.prompt.md`
- `.github/skills/order-flow-audit/SKILL.md`
- `AGENTS.md`
- `SECURITY.md`
- `docs/copilot-context-engineering.html`

## A note on instruction files as a security surface

The files above are read by AI agents and shape what those agents do. That makes them worth
reviewing in a pull request with the same seriousness as executable code, for two reasons.

First, an instruction file can direct an agent toward or away from a behavior, so a malicious
edit is a real attack rather than a documentation typo. Second, `AGENTS.md` and instruction
files are read by Copilot code review, and GitHub reads them from the **pull request head
branch**. A pull request can therefore change the instructions that govern its own review.

Review changes to these files deliberately. That advice applies to your repository as much as
to this one.

## Scope

Findings about GitHub Copilot itself, rather than about the content of this repository, belong
with GitHub. Report those through
[GitHub's security policy](https://github.com/github/.github/blob/main/SECURITY.md).
