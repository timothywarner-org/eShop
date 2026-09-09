# Five prompts to try on this repository

[Return to the lesson](../README.md) · [Repository map](repo-map.md)

**Every prompt here is read-only.** None of them change application code. Run them in VS Code with Copilot Chat, with this repository open at its root so the customizations under `.github/` can be discovered.

Each prompt targets a different rung of the context layer. The point is not the answer. The point is whether you can **defend the answer from the source** afterward.

| # | Prompt | What it exercises | The failure it exposes |
| --- | --- | --- | --- |
| 1 | Cold orientation | Repository instructions | Confident architecture invented from directory names |
| 2 | The ID handoff | Path-scoped Ordering instructions | Conflating a database key with an external identity |
| 3 | Blast radius | The `order-flow-audit` skill | Assuming every contract change breaks every subscriber |
| 4 | Falsify the map | Evidence discipline | Treating documentation as proof |
| 5 | The transfer | Method reuse | Borrowing Ordering's conclusions for a different service |

---

## 1. Cold orientation

**Demonstrates:** repository-wide instructions doing their job. `.github/copilot-instructions.md` requires named services, quoted warnings, and explicit uncertainty. Watch for all three in the answer.

```text
I have never seen this codebase. Name the services a customer request touches
between clicking "checkout" in the web storefront and an order row existing in
the database. Give the file and method for each hop.

For each service-to-service transition, say what carries the data across it:
HTTP, gRPC, or a message on the bus. Do not infer the transport from the
service name.

Then list what you could NOT determine from the source, and name what you
searched for. Do not edit files or run the application.
```

**A good answer** names `WebApp`, `Ordering.API`, and the database, and it distinguishes the gRPC hop to Basket from the HTTP hops to Catalog and Ordering. It admits that the AppHost only proves wiring exists.

**Point at this:** if the answer describes a message queue between the storefront and Ordering without citing a subscription, it inferred the architecture. `src/eShop.AppHost/Program.cs` shows a `WithReference`, and a reference establishes a connection, not a behavior.

---

## 2. The ID handoff

**Demonstrates:** `.github/instructions/ordering.instructions.md` loading automatically because the files under investigation match its `applyTo` glob. Students should open the Copilot response's references and confirm the scoped instruction was applied.

```text
In the order placement flow, two different buyer identifiers move through the
system. One is a numeric database key. One is an external identity string.

Trace both. Show me where each is assigned, which handler consumes it, and
which one ends up on the integration event that leaves the service. Explain
why using the wrong one would be a defect that compiles cleanly.

Quote any REVIEW, TODO, HACK, or FIXME comment in the files you touch,
verbatim. Tell me whether that comment describes a reproduced failure or an
unverified claim. Read-only: do not edit or run anything.
```

**A good answer** separates `Buyer.Id` from `Buyer.IdentityGuid`, notes that `UseHiLo` governs when the numeric key becomes available, and quotes the `REVIEW` comment in `ValidateOrAddBuyerAggregateWhenOrderStartedDomainEventHandler.cs` without smoothing it into a summary.

**Point at this:** the `REVIEW` comment is real source material and it is **not** a reproduced bug report. An answer that says "this is a known bug that causes X in production" has upgraded a warning into a finding. That is the exact move the lesson exists to prevent.

---

## 3. Blast radius

**Demonstrates:** the `order-flow-audit` skill. Copilot should select it on its own from the description, since this request is a change review under `src/Ordering.*`. You can also force it with `/order-flow-audit`.

```text
Suppose I need to add a field to OrderStatusChangedToSubmittedIntegrationEvent
and rename an existing one.

Do not write the change. Instead, enumerate every subscriber to that event
across the repository, and for each one tell me whether it breaks, and why or
why not. Distinguish an additive change from a renaming change.

Then tell me what in this repository would have caught the break: an existing
test, a compiler error, or nothing at all. Be specific about which.
```

**A good answer** finds the subscribers rather than guessing at them, and it separates the additive case from the rename case instead of declaring that all changes break all consumers.

**Point at this:** the last paragraph is the valuable one. "Nothing at all would have caught this" is a legitimate and important answer. An agent that claims a test covers the change should be made to name the test and quote its assertions. A test filename is a lead, not proof of coverage.

---

## 4. Falsify the map

**Demonstrates:** evidence discipline turned against your own context files. This is the prompt that keeps a repository map honest, and it is the one most teams never think to run.

```text
Read docs/repo-map.md. Treat every claim in it as a hypothesis, not as fact.

Pick the four claims that would be most expensive to get wrong. For each one,
open the source it cites and tell me whether the current code still supports
it. Quote the line that confirms or contradicts the claim.

Report in three buckets: Confirmed, Contradicted, and Cannot verify from
source. For anything in the second or third bucket, propose the corrected
wording. Do not edit the file.
```

**A good answer** cites line-level evidence for each verdict and puts runtime behavior into "cannot verify from source" rather than guessing. The map itself says it does not establish live message delivery, key timing in an executed scenario, or price freshness in a browser.

**Point at this:** the map carries an **evidence baseline commit**. Ask the class why that matters. A map without a baseline cannot be audited, because there is no way to know which revision it described.

---

## 5. The transfer

**Demonstrates:** whether the method survives without the worked example. Nothing in the context layer is tuned for Basket, so this is the honest test of transfer.

```text
/trace-flow Where does the price shown next to an item in the shopping basket
come from, and when does it change? Read-only: do not edit files or run the app.

Specifically: is the displayed price stored with the basket, or is it read from
Catalog at render time? If a product price changes in Catalog, what would make
the basket reflect that, and what would leave it stale? Trace the actual data
reads and any event subscriptions before you answer.

Do not reuse conclusions from the Ordering investigation.
```

**A good answer** starts at `BasketState`, crosses into the Basket and Catalog clients, and inspects subscription registrations before claiming that a price-change message updates anything.

**Point at this:** `/trace-flow` is a **prompt file**, and prompt files work in VS Code but not on github.com or in Copilot CLI. If a student is not in VS Code, have them open `.github/prompts/trace-flow.prompt.md` and paste its body. That limitation is a teaching moment about portability, not a bug.

---

## If you have time: run the same question twice

Ask prompt 2 twice. Once in the default agent, once with the **flow-investigator** custom agent selected.

The custom agent is granted only the `search` and `read` tool sets, so the `edit` and `execute` tools are not available to it. The read-only constraint stops being a polite request in the prompt text and becomes a property of the configuration.

| Approach | How "do not edit" is enforced |
| --- | --- |
| Instruction in the prompt | The model is asked to comply |
| Custom agent with restricted `tools` | The capability is absent |

That difference is the whole argument for putting a guardrail in configuration rather than in prose.

---

## What to collect from the session

Finish with one entry in this format, checked against the source before anyone saves it:

```text
Fact: [what the inspected code establishes]
Evidence: [file and method that support the fact]
Reasoning rule: [what to check in similar work, and why]
Scope/home: [where it applies and which existing file should hold it]
```

Navigation facts belong in [the map](repo-map.md). A recurring subsystem check belongs in a path-scoped instruction. A repeatable procedure belongs in the prompt or the skill. Keep unverified hypotheses out of all of them.
