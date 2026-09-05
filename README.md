# eShop, with a GitHub Copilot context engineering layer

[![Upstream](https://img.shields.io/badge/upstream-dotnet%2FeShop-1f6feb?style=flat-square)](https://github.com/dotnet/eShop)
[![License](https://img.shields.io/badge/license-MIT-1f6feb?style=flat-square)](LICENSE)
[![Session](https://img.shields.io/badge/session-Speaking%20Copilot's%20Language-1f6feb?style=flat-square)](docs/copilot-context-engineering.html)

This is a fork of [dotnet/eShop](https://github.com/dotnet/eShop) with one thing added: a
complete, working **context engineering layer** for GitHub Copilot. Upstream's application code
is untouched. Everything below is additive, and the original README follows this section.

Copilot does not read your repository. It **retrieves** from it, and everything it retrieves
competes for the same finite context window as your prompt, your open files, and your chat
history. The files in this fork are an argument about how to spend that budget deliberately
instead of accidentally.

## What was added

Four tiers, six files. The only thing separating the tiers is **when the content is charged
against the context window**.

| Tier | Path | Enters the window when |
| --- | --- | --- |
| **1. Always** | `.github/copilot-instructions.md` | Every request in this repository, no exceptions |
| **1. Always** | `AGENTS.md` | Every request, and also on Copilot code review |
| **2. Conditionally** | `.github/instructions/ordering.instructions.md` | The `applyTo` glob matches a file under `src/Ordering.*` |
| **2. Conditionally** | `.github/instructions/tests.instructions.md` | The `applyTo` glob matches a file under `tests/` |
| **3. On request** | `.github/prompts/trace-flow.prompt.md` | You type `/trace-flow` in chat |
| **4. On relevance** | `.github/skills/order-flow-audit/SKILL.md` | The agent reads the `description` and decides it applies |

Tiers 2 through 4 cost nothing until they are needed. Tier 1 is charged on every single
request, including the ones where its content is irrelevant, which is why both tier-1 files
here are deliberately short. A four-hundred-line `copilot-instructions.md` is a tax you pay
on every question you will ever ask.

## Why the ordering service

The instruction files, the prompt file, and the skill all converge on one real defect that was
already in this codebase before the fork. In
`src/Ordering.API/Application/DomainEventHandlers/ValidateOrAddBuyerAggregateWhenOrderStartedDomainEventHandler.cs`:

```csharp
// REVIEW: The event this creates needs to be sent after SaveChanges has propagated the buyer Id. It currently only
// works by coincidence. If we remove HiLo or if anything decides to yield earlier, it will break.
```

That is the only `REVIEW` comment in the entire `src/Ordering.*` tree, and it documents exactly
the hazard the layer is built to surface: correctness depending on a database-generated Id being
populated first. Every artifact here instructs Copilot to quote that comment verbatim rather
than summarize it away.

Nothing about the demo is synthetic. A prior engineer wrote down where the fragility is, and
the point of a context engineering layer is that the next person to touch the file gets told.

## Try it in about a minute

1. Clone this fork and open it in **VS Code**.
2. Build the workspace index first. See the warning below, because this step is not optional here.
3. Open any file under `src/Ordering.API/` and ask Copilot Chat how an order is placed.
4. Run `/trace-flow` and name the order placement flow.
5. Open the **References** list on the answer. What was retrieved tells you whether a weak answer
   was a retrieval problem or an instruction problem. It answers that question in about ten seconds,
   and it answers it better than switching models does.

> **Build the index before you judge the results.** This repository has roughly **1,150 tracked
> files**. VS Code indexes a workspace automatically only below **750** files, and falls back to a
> basic index using simpler search algorithms above **2,500**. Between those two numbers nothing
> happens until you act, and the outcome is silent either way. Run
> **GitHub Copilot: Build Remote Workspace Index** from the Command Palette, then confirm the tier
> in the Copilot status dashboard in the Status Bar. Evaluating retrieval quality on an unbuilt
> index is the most common way teams reach a wrong conclusion about Copilot.

## Verifying the layer rather than trusting it

A glob that silently matches nothing is indistinguishable from a glob that works. Both of the
`applyTo` patterns here were checked against this tree before they were committed:

- `src/Ordering.API`, `src/Ordering.Domain`, and `src/Ordering.Infrastructure` all exist.
- `tests/` exists, including the `*.FunctionalTests` projects that
  `tests.instructions.md` distinguishes from unit tests.

Do the same on your own repository. Open a file inside the glob and confirm the instructions are
in play, then open one outside it and confirm they are not.

## The session sheet

[`docs/copilot-context-engineering.html`](docs/copilot-context-engineering.html) is the full
reference sheet these files came from, including the three scopes and their precedence, Copilot
Spaces, reading the context budget with `/context` in Copilot CLI, what the remote index will
never see whatever you write, and a sourced citation for every product claim.

GitHub serves that file as source rather than as a rendered page. Open it locally, or enable
**GitHub Pages** on the `docs/` folder to get a shareable URL.

## Relationship to upstream

This fork exists to teach a technique, not to compete with the reference application. It tracks
[dotnet/eShop](https://github.com/dotnet/eShop) and stays under the same **MIT** license.

- Bugs in the eShop application itself belong **upstream**. Please report them there.
- Questions about the context engineering layer belong in this repository's **Issues**.
- Application code here is not modified, so upstream remains the authority on how eShop works.

---

<!-- Upstream dotnet/eShop README follows, unmodified. -->

# eShop Reference Application - "AdventureWorks"

A reference .NET application implementing an e-commerce website using a services-based architecture with [Aspire](https://aspire.dev/).

![eShop Reference Application architecture diagram](img/eshop_architecture.png)

![eShop homepage screenshot](img/eshop_homepage.png)

## Getting Started

This version of eShop is based on .NET 10.

Previous eShop versions:

* [.NET 8](https://github.com/dotnet/eShop/tree/release/8.0)

### Prerequisites

1. Install a [.NET 10 SDK](https://dotnet.microsoft.com/download/dotnet/10.0) that satisfies [`global.json`](global.json).
2. Install the [Aspire CLI](https://aspire.dev/get-started/install-cli/) and verify that it is available:

    ```console
    aspire --version
    ```

3. Install and start an OCI-compatible container runtime. [Docker Desktop](https://www.docker.com/products/docker-desktop/) is the recommended default. [Podman](https://podman.io/docs/installation) is also supported; follow the [Aspire prerequisites](https://aspire.dev/get-started/prerequisites/) to configure it.
4. Clone the repository:

    ```console
    git clone https://github.com/dotnet/eShop.git
    cd eShop
    ```

No separate Aspire workload or Visual Studio component is required; the AppHost SDK and hosting integrations are referenced by the projects in this repository.

#### Optional IDE setup

- [Visual Studio](https://visualstudio.microsoft.com/vs/) with the `ASP.NET and web development` workload.
- [Visual Studio Code with C# Dev Kit](https://code.visualstudio.com/docs/csharp/get-started) and the [Aspire extension](https://aspire.dev/get-started/aspire-vscode-extension/).
- The [.NET MAUI workload](https://learn.microsoft.com/dotnet/maui/get-started/installation) if you want to run the client apps.

### Running the solution

> [!WARNING]
> Ensure that your container runtime is running before starting eShop.

#### From the terminal

From the repository root, run:

```console
aspire run
```

The root [`aspire.config.json`](aspire.config.json) selects `src/eShop.AppHost/eShop.AppHost.csproj`, avoiding ambiguity with the test AppHosts in the repository. When startup completes, the CLI prints a dashboard URL similar to:

```text
Dashboard: https://localhost:<port>/login?t=<token>
```

Press <kbd>Ctrl</kbd>+<kbd>C</kbd> to stop the AppHost. See the [`aspire run` command](https://aspire.dev/reference/cli/commands/aspire-run/) for additional options.

To run the AppHost in the background instead:

```console
aspire start
aspire ps
```

When you are finished, run `aspire stop`. See the [`aspire start` command](https://aspire.dev/reference/cli/commands/aspire-start/) for details.

#### From Visual Studio

1. Open `eShop.Web.slnf`.
2. Set `src/eShop.AppHost/eShop.AppHost.csproj` as the startup project.
3. Press <kbd>Ctrl</kbd>+<kbd>F5</kbd> to start eShop and open the Aspire dashboard.

### Running tests

Run the server tests:

```powershell
dotnet test --solution eShop.Web.slnf
```

Run the Playwright browser journeys. Playwright starts the AppHost automatically, so ensure your container runtime is running first.

```powershell
npm ci
npx playwright install chromium
npm run test:e2e
```

### Optional: AI Chatbot with Microsoft Foundry

This option provisions a Microsoft Foundry resource during local development, so first authenticate to Azure and configure the subscription and location:

```powershell
az login
aspire secret set "Azure:SubscriptionId" "<subscription-id>"
aspire secret set "Azure:Location" "eastus"
```

Then enable Foundry and start eShop:

```powershell
$env:UseFoundry = "true"
aspire run
```

Aspire provisions the `gpt-4.1-mini` and `text-embedding-3-small` deployments and injects their connection information into the consuming projects. The Foundry hosting integration currently uses a preview package. See [local Azure provisioning](https://aspire.dev/integrations/cloud/azure/local-provisioning/) and the [Microsoft Foundry hosting integration](https://aspire.dev/integrations/cloud/azure/azure-ai-foundry/azure-ai-foundry-host/) for details.

### Deploy to Azure Container Apps

The AppHost is already configured with an Azure Container Apps environment, so the Aspire CLI can deploy directly from the application model. See the [Aspire Azure Container Apps deployment guide](https://aspire.dev/deployment/azure/container-apps/) for details.

> [!WARNING]
> This sample deploys PostgreSQL, Redis, and RabbitMQ as containers in Azure Container Apps. This configuration is intended for evaluation and demonstrations, not production data.

Prerequisites:

- The prerequisites listed above, including a running container runtime.
- The [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli), an active Azure subscription, and permission to create resources.

Sign in, optionally preview the deployment pipeline, and deploy:

```console
az login
aspire deploy --list-steps
aspire deploy
```

For local interactive use, `aspire deploy` prompts for missing Azure settings. For non-interactive use, provide them explicitly:

```powershell
$env:Azure__SubscriptionId = "<subscription-id>"
$env:Azure__Location = "eastus"
$env:Azure__ResourceGroup = "rg-eshop-demo"
aspire deploy --non-interactive
```

Use [`aspire publish`](https://aspire.dev/reference/cli/commands/aspire-publish/) when you need deployment artifacts for inspection or another deployment tool. Running it first is not required: `aspire deploy` invokes the deployment pipeline and its dependencies directly rather than consuming an earlier publish output.

When you no longer need the deployment, run [`aspire destroy`](https://aspire.dev/reference/cli/commands/aspire-destroy/). This deletes the entire configured resource group, including resources that Aspire did not create, so review the target carefully before confirming.

## Contributing

For more information on contributing to this repo, read [the contribution documentation](./CONTRIBUTING.md) and [the Code of Conduct](CODE-OF-CONDUCT.md).

### Sample data

The sample catalog data is defined in [catalog.json](https://github.com/dotnet/eShop/blob/main/src/Catalog.API/Setup/catalog.json). Those product names, descriptions, and brand names are fictional and were generated using [GPT-35-Turbo](https://learn.microsoft.com/en-us/azure/ai-services/openai/how-to/chatgpt), and the corresponding [product images](https://github.com/dotnet/eShop/tree/main/src/Catalog.API/Pics) were generated using [DALL·E 3](https://openai.com/dall-e-3).

## eShop on Azure

For a version of this app configured for deployment on Azure, please view [the eShop on Azure](https://github.com/Azure-Samples/eShopOnAzure) repo.
