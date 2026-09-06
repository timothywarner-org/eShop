# Repository map: choose where to investigate

**Use this map to find a starting point, then read the code to establish what happens.** It covers the web storefront and the lesson's two investigations. It is deliberately selective; a directory listing alone cannot explain a system.

**Evidence baseline:** source inspected on September 6, 2026, at commit [`edf3daf`](https://github.com/timothywarner-org/eShop/tree/edf3dafe882f67024309ac8f7d0d17c1e978adf1). These are source observations, not results from running eShop. Recheck relevant entries when working on another revision. [Return to the student walkthrough](../README.md).

## Where to start

| Your question | Start here | What the source establishes |
| --- | --- | --- |
| What runs, and what is wired together? | [AppHost `Program.cs`](../src/eShop.AppHost/Program.cs) | Aspire declares services, Redis, RabbitMQ, and PostgreSQL databases. A resource reference establishes wiring; inspect a caller or subscription to establish behavior. |
| How does the storefront reach services? | [WebApp registrations](../src/WebApp/Extensions/Extensions.cs) | A gRPC client targets Basket; HTTP clients target Catalog and Ordering. Order-status event subscriptions are registered separately. |
| Where do basket contents and displayed values come from? | [WebApp `BasketState`](../src/WebApp/Services/BasketState.cs), [Basket client](../src/WebApp/Services/BasketService.cs), [Catalog client](../src/WebAppComponents/Services/CatalogService.cs) | `BasketState` coordinates reads and checkout. Follow these methods to distinguish stored quantities, product details, and cached UI state. |
| How are basket quantities stored? | [Basket gRPC service](../src/Basket.API/Grpc/BasketService.cs), [Redis repository](../src/Basket.API/Repositories/RedisBasketRepository.cs), [registrations](../src/Basket.API/Extensions/Extensions.cs) | gRPC operations use the repository; the registered implementation uses Redis. Event subscriptions are declared in the registrations file. |
| Where are product updates handled? | [Catalog endpoints](../src/Catalog.API/Apis/CatalogApi.cs), [Catalog registrations](../src/Catalog.API/Extensions/Extensions.cs) | Catalog handles item reads and updates, registers its database context, and subscribes to selected order events. Trace publication and subscribers separately. |
| Where does order placement enter domain logic? | [Orders endpoints](../src/Ordering.API/Apis/OrdersApi.cs), [create-order handler](../src/Ordering.API/Application/Commands/CreateOrderCommandHandler.cs) | The endpoint sends an identified command through MediatR; its create-order handler builds the aggregate and invokes the unit of work. Follow the detailed path below. |
| What continues work after the request? | [OrderProcessor registrations](../src/OrderProcessor/Extensions/Extensions.cs), [PaymentProcessor entry point](../src/PaymentProcessor/Program.cs), [Webhooks registrations](../src/Webhooks.API/Extensions/Extensions.cs) | OrderProcessor registers a hosted grace-period service. PaymentProcessor subscribes to stock-confirmed events. Webhooks registers price-change and selected order-status subscribers. These are distinct paths to investigate. |
| Where are authentication and shared behavior configured? | [WebApp authentication](../src/WebApp/Extensions/Extensions.cs), [Identity entry point](../src/Identity.API/Program.cs), [service defaults](../src/eShop.ServiceDefaults) | Authentication configuration and common service behavior live outside the business handlers. Inspect them when identity or cross-cutting behavior affects the question. |

## Follow one flow

### Order placement: numeric IDs and buyer identity

Start with [WebApp `OrderingService.CreateOrder`](../src/WebApp/Services/OrderingService.cs), then follow:

1. **HTTP entry:** [Orders API](../src/Ordering.API/Apis/OrdersApi.cs) receives the request and sends an identified command through MediatR.
2. **Aggregate creation:** [create-order handler](../src/Ordering.API/Application/Commands/CreateOrderCommandHandler.cs) creates an [Order](../src/Ordering.Domain/AggregatesModel/OrderAggregate/Order.cs), whose constructor queues `OrderStartedDomainEvent`.
3. **Domain-event dispatch:** [OrderingContext](../src/Ordering.Infrastructure/OrderingContext.cs) dispatches domain events before `SaveChangesAsync`; [the dispatcher](../src/Ordering.Infrastructure/MediatorExtension.cs) publishes the queued events through MediatR.
4. **Buyer/payment verification:** [buyer validation handler](../src/Ordering.API/Application/DomainEventHandlers/ValidateOrAddBuyerAggregateWhenOrderStartedDomainEventHandler.cs) calls [Buyer](../src/Ordering.Domain/AggregatesModel/BuyerAggregate/Buyer.cs), which queues `BuyerAndPaymentMethodVerifiedDomainEvent`. The [order-update handler](../src/Ordering.API/Application/DomainEventHandlers/UpdateOrderWhenBuyerAndPaymentMethodVerifiedDomainEventHandler.cs) uses `Buyer.Id` and `Payment.Id` to set the order's references.
5. **Persistence and integration messaging:** inspect [buyer mapping](../src/Ordering.Infrastructure/EntityConfigurations/BuyerEntityTypeConfiguration.cs) and [payment mapping](../src/Ordering.Infrastructure/EntityConfigurations/PaymentMethodEntityTypeConfiguration.cs) for `UseHiLo`. The buyer validation handler supplies `buyer.IdentityGuid` to the submitted integration event. [TransactionBehavior](../src/Ordering.API/Application/Behaviors/TransactionBehavior.cs) commits the transaction before publishing its logged integration events through the event bus.

**Reasoning rule:** distinguish a numeric database key from an external identity, and distinguish in-process domain-event dispatch from integration-event publication. Trace value assignment and transaction boundaries before claiming that changing execution order causes a defect. The `REVIEW` comment identifies a dependency to investigate; it does not reproduce a failure.

### Basket display: transfer the method

Investigate **where the displayed price comes from and when it refreshes**. Begin at `BasketState.GetBasketItemsAsync`, follow its data reads, and locate cache creation and invalidation. Use the Basket and Catalog clients above to cross the service boundaries. Inspect event registration and handlers before claiming a price-change message updates the basket.

Record the source path, one verified fact, and one reasoning rule. Explain what would require runtime observation. The map gives you the entry points; your trace supplies the conclusion.

## Where to look for tests

| Question to check | Existing test starting point |
| --- | --- |
| Buyer and order aggregate behavior | [BuyerAggregateTest](../tests/Ordering.UnitTests/Domain/BuyerAggregateTest.cs), [OrderAggregateTest](../tests/Ordering.UnitTests/Domain/OrderAggregateTest.cs) |
| Create-order command behavior | [NewOrderCommandHandlerTest](../tests/Ordering.UnitTests/Application/NewOrderCommandHandlerTest.cs) |
| Ordering API behavior | [OrderingApiTests](../tests/Ordering.FunctionalTests/OrderingApiTests.cs) |
| Basket service and persistence behavior | [BasketServiceTests](../tests/Basket.UnitTests/BasketServiceTests.cs), [RedisBasketRepositoryTests](../tests/Basket.FunctionalTests/RedisBasketRepositoryTests.cs) |
| Catalog API behavior | [CatalogApiTests](../tests/Catalog.FunctionalTests/CatalogApiTests.cs) |

**A test filename is a lead, not proof of coverage.** Read the setup and assertions to see whether they exercise your claim. Test execution is outside this source-reading lesson; see [test setup](../tests/README.md) and the [application guide](eshop-application.md) when runtime verification is needed.

## Create a map for your own repository

Use this prompt with an LLM that can inspect your checkout:

```text
Draft a concise repository map for a newcomer investigating one business flow.
Inspect files without editing them or running the application.

Start with the actual directory structure, entry points, and service registrations.
For the major areas relevant to the flow, give their responsibilities and source paths.
Follow callers, handlers, and registrations to verify relationships. Label each
boundary's transport. Do not infer architecture from names or directory layout alone.
Include one flow to trace, relevant test entry points, the inspected commit,
and explicit unknowns. Describe what was inspected, not the entire repository.

Finish with up to two verified facts and their evidence. For each, preserve the
reasoning rule that should guide future investigations, explain why it matters,
and state where it applies. Keep hypotheses out of permanent instructions.
Propose the map in chat for review; do not save it yet.

Business flow: [name the action you want to understand]
```

Review the paths and relationships, then save the checked map as ordinary documentation. **`docs/repo-map.md` is not a special Copilot instruction filename.** In this fork, `AGENTS.md` and `/trace-flow` tell the agent when to read it. Check the tool activity or explicitly attach the file when you need to confirm it was used. [VS Code custom-instruction guidance](https://code.visualstudio.com/docs/agent-customization/custom-instructions)

## Keep the map useful

- **Preserve the fact and the reasoning rule.** Put navigation facts and source links here; put a short recurring check in the relevant scoped instruction or reusable prompt. See the [worked example](../README.md#preserve-the-fact-and-the-reasoning-rule).
- **Update what changed.** When entry points, contracts, registrations, or responsibilities change, recheck affected entries and record the new evidence baseline in the same change. Remove obsolete claims.
- **Keep the limits visible.** This map does not establish live message delivery, database key timing in an executed scenario, price freshness in a browser, or test coverage. Mobile/hybrid clients, optional AI features, and deployment details are outside its inspected flows.
- **Keep it small.** Add a subsystem map only when the top-level map becomes difficult to use. Do not duplicate source code or turn the always-on instruction file into an architecture manual.
