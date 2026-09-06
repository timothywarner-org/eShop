# Speaking Copilot's Language

**Understand an unfamiliar codebase, verify what you learn, and give GitHub Copilot reusable context for the next investigation.**

This is Tim Warner's teaching fork of [dotnet/eShop](https://github.com/dotnet/eShop), a .NET application with multiple services. It gives us a real system to investigate: commands, domain events, database operations, and messages crossing service boundaries. You do not need to understand all of eShop before you begin.

The lesson follows one loop: **explore → verify → preserve → reuse**. Copilot helps you find and explain code. You check its explanation against the source, then record the useful knowledge in instructions, prompts, and skills that travel with the repository.

**Start here:** [Try the investigation](#try-the-investigation) · [Read the HTML leave-behind](docs/copilot-context-engineering.html) · [See the context files](#the-context-layer) · [Run eShop](docs/eshop-application.md)

## What you will learn

By the end, you should be able to:

- **Trace one business flow** through an unfamiliar application using file and method references.
- **Distinguish evidence from inference**, including existing warnings, assumptions, and unanswered questions.
- **Choose where knowledge belongs**: repository instructions, path-scoped instructions, a reusable prompt, or an agent skill.
- **Improve the next investigation** without making every request carry an entire architecture manual.

The examples use C#, but the method applies to any language. Familiarity with functions, services, and database writes is enough to follow the investigation.

## Why context matters

Copilot builds an answer from the context available to it. In VS Code, an agent can search by meaning, search for exact text, follow symbol references, and read files. It can investigate repeatedly, but a fluent answer still needs checking. Your question, instructions, conversation history, and tool results share a finite context window. [How workspace context works](https://code.visualstudio.com/docs/agents/reference/workspace-context)

Useful context answers two questions: **Where should we investigate? What should we check when we get there?** This fork demonstrates both. Its instructions require evidence, existing warnings, and explicit uncertainty, while its prompt defines a repeatable investigation.

These files do not train the model or make it remember every file. They preserve guidance that supported tools can load into future requests.

## Try the investigation

**You can read the lesson and inspect all the source on GitHub without installing anything.** To try the prompts yourself, use VS Code with GitHub Copilot Chat enabled for your account. Open the repository root so its customizations can be discovered.

You do **not** need to build eShop, start Docker, provision Azure resources, or run its services for this source-reading exercise. Full application setup is [a separate guide](docs/eshop-application.md).

### 1. Open this fork

Clone it with Git, or download the repository ZIP from GitHub and extract it. For PowerShell users:

```powershell
# Use the teaching fork so the lesson's instructions and prompt are included.
git clone https://github.com/timothywarner-org/eShop.git
if ($LASTEXITCODE -ne 0) { throw 'Clone failed; check Git access before continuing.' }
Set-Location -LiteralPath eShop -ErrorAction Stop
```

Open that folder in VS Code. Start a fresh Copilot chat. Check the indexing status if semantic search is unavailable; agents can also use text search and file reads while an index is being prepared. There is no file-count threshold you need to memorize for this lesson. [Current indexing guidance](https://code.visualstudio.com/docs/agents/reference/workspace-context#semantic-search)

### 2. Investigate one question

Paste this into Copilot Chat:

```text
Investigate the order-placement flow without changing files or running the application.
When a buyer and payment method are created or verified, how do their numeric IDs
reach the order? What does that depend on, and how does it differ from the buyer
identity carried by the submitted integration event?

Trace the relevant files and methods. Quote existing REVIEW, TODO, HACK, or FIXME
comments. Separate what the code proves from assumptions and anything you could
not determine. Do not propose a fix yet.
```

Read the response, then open its cited files and any available References or tool details. **A reference is a starting point for verification, not proof that the explanation is correct.**

### 3. Check the explanation against the code

The following files provide the evidence for this investigation. Read them after your first attempt if you want to find the path yourself.

| Evidence | What to inspect |
| --- | --- |
| [Buyer validation handler](src/Ordering.API/Application/DomainEventHandlers/ValidateOrAddBuyerAggregateWhenOrderStartedDomainEventHandler.cs) | The existing `REVIEW` comment, payment verification, adding a new buyer, saving entities, and creating the integration event. |
| [Buyer aggregate](src/Ordering.Domain/AggregatesModel/BuyerAggregate/Buyer.cs) | `VerifyOrAddPaymentMethod` queues a domain event carrying buyer and payment objects. |
| [Ordering context](src/Ordering.Infrastructure/OrderingContext.cs) | `SaveEntitiesAsync` dispatches domain events before calling `SaveChangesAsync`. |
| [Order-update handler](src/Ordering.API/Application/DomainEventHandlers/UpdateOrderWhenBuyerAndPaymentMethodVerifiedDomainEventHandler.cs) | `SetPaymentMethodVerified` receives `Buyer.Id` and `Payment.Id`. |
| [Buyer mapping](src/Ordering.Infrastructure/EntityConfigurations/BuyerEntityTypeConfiguration.cs) and [payment mapping](src/Ordering.Infrastructure/EntityConfigurations/PaymentMethodEntityTypeConfiguration.cs) | Both numeric IDs use `UseHiLo`; investigate when values become available instead of assuming every key is assigned at save time. |
| [Submitted integration event](src/Ordering.API/Application/IntegrationEvents/Events/OrderStatusChangedToSubmittedIntegrationEvent.cs) | Its buyer field is `BuyerIdentityGuid`, supplied from `buyer.IdentityGuid`, rather than the numeric `Buyer.Id`. |

The warning is real source material, but **a warning comment is a claim to investigate**. Finding it does not reproduce a runtime failure or prove every part of its explanation. Follow the implementation, state the dependency, and identify what would need a test.

### 4. Repeat with a reusable method

In a fresh chat, invoke the included prompt:

```text
/trace-flow Order placement, focusing on buyer/payment verification and the order's
numeric foreign keys. Read-only investigation: do not edit files or execute the app.
```

The [prompt file](.github/prompts/trace-flow.prompt.md) asks for five sections: **Path, Boundaries, Order dependencies, Existing warnings, and What I could not determine**. If the command is not available in your client, open the file and paste its body into chat, followed by the flow and the read-only constraint. [Prompt-file documentation](https://code.visualstudio.com/docs/agent-customization/prompt-files)

Compare the answers using evidence, not length:

| Check | A useful answer demonstrates |
| --- | --- |
| Traceability | Its files and methods exist and support the described steps. |
| Identity distinction | Numeric database keys are distinguished from the buyer's identity string. |
| Execution order | It explains when events are queued, dispatched, and followed by persistence. |
| Existing warnings | It quotes the relevant comment and checks its claim against the implementation. |
| Uncertainty | It names unresolved questions and separates source analysis from runtime verification. |

**This fork already includes the context layer.** Both attempts can use it. This exercise compares an ordinary question with a structured investigation; it is not an instruction-free baseline or proof that one model is better. A useful first answer is a success, not a failed demonstration.

## The context layer

These six files are the teaching layer. Other skills inherited from upstream support application development and are outside the core exercise.

| Mechanism | File | Purpose in this lesson |
| --- | --- | --- |
| **Repository instructions** | [.github/copilot-instructions.md](.github/copilot-instructions.md) | Establish the stack and require source-grounded explanations, warnings, and explicit uncertainty. |
| **Agent conventions** | [AGENTS.md](AGENTS.md) | Define build/test conventions, edit boundaries, and the canonical lesson sources. |
| **Path-scoped instructions** | [ordering.instructions.md](.github/instructions/ordering.instructions.md) | Focus Ordering investigations on aggregates, ID availability, and transaction boundaries. |
| **Path-scoped instructions** | [tests.instructions.md](.github/instructions/tests.instructions.md) | Distinguish this repository's unit and functional test conventions. |
| **Reusable prompt** | [trace-flow.prompt.md](.github/prompts/trace-flow.prompt.md) | Make the investigation method repeatable through `/trace-flow`. |
| **Agent skill** | [order-flow-audit/SKILL.md](.github/skills/order-flow-audit/SKILL.md) | Package the Ordering review procedure for relevant investigations and changes. |

Think in terms of **when content is needed**. Keep repository guidance concise, scope subsystem rules to the relevant paths, invoke prompts for repeatable tasks, and use skills for procedures. Skills expose discovery metadata before their full instructions load. Client support, settings, and policies affect behavior, so verify which customizations were used. [Custom instructions](https://code.visualstudio.com/docs/agent-customization/custom-instructions), [agent skills](https://code.visualstudio.com/docs/agent-customization/agent-skills)

## Apply the method to a codebase you do not know

You do not need an architecture manual before you start:

1. **Explore:** choose one user action and ask Copilot for its path through the code.
2. **Verify:** open the cited files, follow the calls, and challenge missing or contradictory evidence.
3. **Preserve:** write a short instruction containing only the conventions and dependencies you verified, scoped to the relevant files.
4. **Reuse:** investigate a different flow and check whether the guidance helps without steering Copilot toward an unsupported answer.

For a transfer exercise, use `/trace-flow` to investigate **where the basket's displayed price comes from and when it is refreshed**. Find the data reads, caching, and any relevant event subscriptions without assuming the update mechanism. Reuse the evidence standards, but derive that flow's facts from its own source. You have learned the method when you can explain a flow that this README has not already explained for you.

## The HTML leave-behind

**[docs/copilot-context-engineering.html](docs/copilot-context-engineering.html)** is the canonical reference handout. It includes the six copyable files, context-management guidance, optional Copilot Spaces and CLI material, and links to product documentation.

GitHub displays HTML source rather than running it. After cloning or downloading this repository, open the file in a browser. It is self-contained for offline reading; its documentation links need an internet connection. Browser printing is supported. You can also download the raw HTML file from GitHub and open it locally.

Spaces and Copilot CLI are extensions to the lesson. Neither is required for the investigation above.

## Canonical sources and contributions

**This is the canonical student and demo repository, `timothywarner-org/eShop`.** Maintain the README, context files, and HTML handout here. Distributed handouts are copies of the repository version, not separate sources to edit. Instructor scripts, private notes, and rehearsal recordings are maintained separately and are not published here.

- Update a context file and its copyable HTML example together.
- Ground code claims in the checked-out source and volatile product claims in current first-party documentation.
- Keep public materials suitable for any student. Exclude client identities, private meeting details, and internal adoption data.
- Use [this fork's Issues](https://github.com/timothywarner-org/eShop/issues) for lesson questions, broken instructions, or documentation corrections. Include the prompt, relevant files, expected behavior, and observed behavior.

The application comes from [dotnet/eShop](https://github.com/dotnet/eShop) under the [MIT license](LICENSE). The lesson adds context and learning materials; it does not repair the Ordering behavior discussed here. See the [application guide](docs/eshop-application.md) for setup and runtime work, and follow [upstream contribution guidance](https://github.com/dotnet/eShop/blob/main/CONTRIBUTING.md) for application changes.

**Tim Warner** · [TechTrainerTim.com](https://techtrainertim.com) · [Pluralsight author page](https://www.pluralsight.com/authors/tim-warner)
