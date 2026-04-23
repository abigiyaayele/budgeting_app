# budgeting_app
AI-powered budgeting app using NestJS, PostgreSQL, Clean Architecture, and DDD.
<img width="1472" height="1960" alt="image" src="https://github.com/user-attachments/assets/eef91a98-72d8-45eb-9623-4ab00567ba38" />
# Project structure
```
src/
├── modules/
│   ├── budget/
│   │   ├── domain/
│   │   │   ├── budget.aggregate.ts        # Aggregate root
│   │   │   ├── value-objects/             # Money, Period, BudgetId
│   │   │   ├── events/                    # BudgetExceededEvent
│   │   │   ├── services/                  # SpendingAnalyzer (domain svc)
│   │   │   └── repositories/              # IBudgetRepository (interface/port)
│   │   ├── application/
│   │   │   ├── commands/                  # CreateBudgetCommand + Handler
│   │   │   ├── queries/                   # GetBudgetStatusQuery + Handler
│   │   │   └── dto/                       # Input/output DTOs
│   │   ├── infrastructure/
│   │   │   ├── persistence/               # TypeORM entity + repo adapter
│   │   │   └── budget.module.ts
│   │   └── presentation/
│   │       └── budget.controller.ts
│   ├── transaction/                        # Same structure
│   ├── goal/                               # Same structure
│   ├── user/                               # Same structure
│   └── ai/
│       ├── insights/                       # Spending insights service
│       ├── forecast/                       # Budget forecasting service
│       ├── chat/                           # Chat assistant + SSE streaming
│       └── smart/                          # Auto-categorize, OCR, nudges
├── shared/
│   ├── domain/                             # AggregateRoot, DomainEvent, ValueObject base classes
│   ├── infrastructure/                     # TypeORM base repo, event bus
│   └── guards/                             # JWT guard, RBAC
├── database/
│   ├── migrations/
│   └── seeds/
└── main.ts
```
#  Technology stack
Core: NestJS · TypeScript · PostgreSQL · TypeORM · @nestjs/cqrs
AI features: OpenAI or Anthropic SDK · LangChain (optional for RAG) · @nestjs/schedule for digest jobs
Infrastructure: Redis (cache + rate limiting) · Bull (job queues) · Passport + JWT · class-validator
Observability: OpenTelemetry · Pino logger · Sentry

## Key DDD patterns to implement
Aggregates enforce all invariants internally — Budget.addTransaction() checks limits before raising a BudgetExceededEvent. Nothing outside the aggregate modifies its state directly.
Value objects like Money carry validation (no negative amounts, currency code required) and equality by value, not reference.
Repository interfaces live in the domain layer. TypeORM implementations live in infrastructure. The domain never imports TypeORM.
Domain events flow out of aggregates, get picked up by @EventsHandler in the application layer, and trigger side effects like notifications or AI insight recalculation.
CQRS separates writes (commands mutate state via the aggregate) from reads (queries can hit optimized read models or materialized views directly).

## AI features in detail
Chat assistant — inject the user's last 90 days of spending as context, stream the LLM response back over SSE. Cache results in Redis with a TTL so repeated similar questions don't burn tokens.
Spending insights — run nightly via @Cron. Detect anomalies (>2σ from category mean), surface category-level tips, and queue a weekly digest email.
Budget forecast — use a simple exponential smoothing model (or an LLM prompt with historical data) to estimate end-of-month spend per category and project goal ETAs.
Auto-categorize — on TransactionCreated event, call a classification prompt with merchant name + amount → category. Falls back to regex rules if the AI is unavailable.
Receipt OCR — upload image → S3, run through a vision model (GPT-4o or Claude's vision), extract merchant/amount/date, create a transaction draft for user confirmation.
