---
name: rust-ddd
description: Guide Rust projects using pragmatic Domain-Driven Design (DDD). Use when (1) starting new Rust projects needing architecture guidance, (2) refactoring existing Rust code toward DDD, (3) deciding how to organize domain logic with frameworks like axum, sea-orm, redis, message queues. Provides tiered complexity levels matching project scale.
metadata:
  version: "1.2"
---

# Rust DDD

Pragmatic DDD for Rust. Match complexity to project scale.

## Complexity Tiers Overview

| Tier | Name | Entities | Key Concepts |
|------|------|----------|--------------|
| 1 | Layered | 5-10 | Domain/Infra separation, repository traits |
| 2 | Modular | 10-20 | Application layer, commands/queries |
| 3 | Tactical DDD | 20-40 | Aggregates, value objects, domain events |
| 4 | CQRS | 40-60 | Separate read/write models, event sourcing |
| 5 | Strategic DDD | 60+ | Bounded contexts, integration events, sagas |

**Recommendation:** Prefer Tier 2/3 for most projects. Even when project scale slightly exceeds the suggested entity ranges, favor staying at Tier 2/3 rather than prematurely upgrading. The complexity cost of higher tiers often outweighs the benefits until clear upgrade signals appear.

## Complexity Tiers

### Tier 1: Layered

```
.
├── Cargo.toml              # [bin] single crate
└── src/
    ├── domain/
    │   ├── entities.rs      # Pure domain types
    │   ├── repositories.rs  # Repository traits
    │   └── services.rs      # Business logic
    ├── infra/
    │   ├── persistence.rs   # sea-orm implementations
    │   └── http.rs          # axum handlers
    └── main.rs
```

**Key concepts:**
- Separate domain types from ORM entities
- Repository traits in domain, implementations in infra
- Business logic in domain services

**Use when:** 5-10 entities, small team, need testable domain logic.

### Tier 2: Modular

```
.
├── Cargo.toml              # [bin] single crate
└── src/
    ├── domain/
    │   ├── entities/
    │   ├── repositories.rs  # Traits
    │   └── services.rs
    ├── application/
    │   ├── commands/        # Write operations
    │   ├── queries/         # Read operations
    │   └── handlers.rs
    ├── infra/
    │   ├── persistence/
    │   └── http/
    └── main.rs
```

**Key concepts:**
- Application layer orchestrates domain logic
- Command/Query separation (not full CQRS)
- Handlers coordinate repositories and services

**Use when:** 10-20 entities, multiple developers, need clear use case boundaries.

### Tier 3: Tactical DDD

```
.
├── Cargo.toml              # [bin] single crate
└── src/
    ├── domain/
    │   ├── aggregates/
    │   │   ├── order/
    │   │   │   ├── mod.rs
    │   │   │   ├── order.rs       # Aggregate root
    │   │   │   ├── order_item.rs  # Entity
    │   │   │   └── events.rs      # Domain events
    │   │   └── customer/
    │   ├── value_objects/
    │   ├── repositories.rs
    │   └── services.rs
    ├── application/
    │   ├── commands/
    │   ├── queries/
    │   └── event_handlers.rs
    ├── infra/
    │   ├── persistence/
    │   ├── messaging/
    │   └── http/
    └── main.rs
```

**Key concepts:**
- Aggregates enforce consistency boundaries
- Value objects for immutable domain concepts
- Domain events for side effects and decoupling
- Internal event bus for in-process events

**Use when:** 20-40 entities, complex invariants, need event-driven decoupling.

### Tier 4: CQRS

```
.
├── Cargo.toml              # [bin] single crate
└── src/
    ├── domain/
    │   ├── aggregates/
    │   ├── events/
    │   └── repositories.rs      # Write-side only
    ├── application/
    │   ├── commands/
    │   │   ├── handlers.rs
    │   │   └── validators.rs
    │   └── queries/
    │       ├── handlers.rs
    │       └── read_models.rs   # Optimized projections
    ├── infra/
    │   ├── write_store/         # Event store / write DB
    │   ├── read_store/          # Denormalized read DB
    │   ├── projections/         # Event → read model
    │   └── http/
    └── main.rs
```

**Key concepts:**
- Separate write model (aggregates) from read model (projections)
- Event store as source of truth (optional)
- Projections build optimized read models
- Eventual consistency between write and read

**Use when:** 40-60 entities, read/write performance disparity, need audit trail.

### Tier 5: Strategic DDD

```
.
├── Cargo.toml                   # [workspace]
└── crates/
    ├── shared-kernel/           # [lib] shared types across contexts
    │   ├── Cargo.toml
    │   └── src/
    ├── contexts/
    │   ├── ordering/            # [lib] bounded context
    │   │   ├── Cargo.toml
    │   │   └── src/
    │   │       ├── domain/
    │   │       ├── application/
    │   │       ├── infra/
    │   │       └── api/
    │   ├── inventory/           # [lib] bounded context
    │   │   └── Cargo.toml
    │   ├── payment/             # [lib] bounded context
    │   │   └── Cargo.toml
    │   └── shipping/            # [lib] bounded context
    │       └── Cargo.toml
    ├── integration-events/      # [lib] cross-context contracts
    │   ├── Cargo.toml
    │   └── src/
    ├── saga-orchestrator/       # [lib] distributed transactions
    │   ├── Cargo.toml
    │   └── src/
    ├── api-gateway/             # [bin] HTTP entrypoint
    │   ├── Cargo.toml
    │   └── src/
    └── platform/
        ├── observability/       # [lib] tracing, metrics
        │   └── Cargo.toml
        └── messaging/           # [lib] event bus infrastructure
            └── Cargo.toml
```

**Key concepts:**
- Bounded contexts with independent models
- Integration events for cross-context communication
- Saga pattern for distributed transactions
- Context mapping (ACL, shared kernel, etc.)

**Use when:** 60+ entities, multiple teams, independent deployment, distributed transactions.

## Framework Roles

| Framework | Role | Layer |
|-----------|------|-------|
| axum | HTTP Adapter | Infrastructure |
| sea-orm | Repository impl | Infrastructure |
| redis | Cache/Event store | Infrastructure |
| lapin/rdkafka | Event publisher | Infrastructure |
| tonic | gRPC Adapter | Infrastructure |
| tokio-cron-scheduler | Scheduled tasks | Infrastructure |

## Adapter Placement

### Inbound (Serving APIs)

| Type | Location |
|------|----------|
| HTTP | `infra/http/` |
| gRPC Server | `infra/grpc/server.rs` |
| Message Consumer | `infra/messaging/consumer.rs` |
| Scheduled Job | `infra/scheduler/jobs.rs` |

```rust
// infra/grpc/server.rs
pub struct OrderGrpcService<H: OrderHandler> { handler: H }

#[tonic::async_trait]
impl OrderService for OrderGrpcService<H> {
    async fn create_order(&self, req: Request<CreateOrderReq>) -> Result<Response<OrderResp>, Status> {
        let id = self.handler.handle(req.into_inner().into()).await?;
        Ok(Response::new(OrderResp { id: id.to_string() }))
    }
}
```

```rust
// infra/scheduler/jobs.rs
use tokio_cron_scheduler::{Job, JobScheduler};

pub struct OrderCleanupJob<H: CleanupExpiredOrdersHandler> {
    handler: Arc<H>,
}

impl<H: CleanupExpiredOrdersHandler> OrderCleanupJob<H> {
    pub fn new(handler: Arc<H>) -> Self {
        Self { handler }
    }

    pub async fn schedule(self, scheduler: &JobScheduler) -> Result<(), SchedulerError> {
        let handler = self.handler.clone();

        // Run every day at 2 AM
        let job = Job::new_async("0 0 2 * * *", move |_uuid, _lock| {
            let handler = handler.clone();
            Box::pin(async move {
                if let Err(e) = handler.handle().await {
                    eprintln!("Failed to cleanup expired orders: {}", e);
                }
            })
        })?;

        scheduler.add(job).await?;
        Ok(())
    }
}

// main.rs - setup scheduler
#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let scheduler = JobScheduler::new().await?;

    // Setup handlers
    let cleanup_handler = Arc::new(CleanupExpiredOrdersHandler::new(order_repo));

    // Register jobs
    OrderCleanupJob::new(cleanup_handler).schedule(&scheduler).await?;

    // Start scheduler
    scheduler.start().await?;

    // ... rest of application setup
}
```

### Outbound (Calling External)

| Type | Trait | Impl |
|------|-------|------|
| 3rd Party API | `domain/ports.rs` | `infra/external/` |
| gRPC Client | `domain/ports.rs` | `infra/grpc/clients/` |
| Database | `domain/repositories.rs` | `infra/persistence/` |

```rust
// domain/ports.rs
#[async_trait]
pub trait PaymentGateway {
    async fn charge(&self, amount: Money, card: CardToken) -> Result<PaymentId, PaymentError>;
}

// infra/external/stripe.rs
impl PaymentGateway for StripeGateway {
    async fn charge(&self, amount: Money, card: CardToken) -> Result<PaymentId, PaymentError> {
        // call stripe API
    }
}
```

```rust
// domain/ports.rs - calling other microservices
#[async_trait]
pub trait InventoryService {
    async fn reserve(&self, items: Vec<ItemId>) -> Result<ReservationId, InventoryError>;
}

// infra/grpc/clients/inventory.rs
impl InventoryService for InventoryGrpcClient {
    async fn reserve(&self, items: Vec<ItemId>) -> Result<ReservationId, InventoryError> {
        self.client.reserve_items(items.into()).await?.into()
    }
}
```

### Tier 2+ Structure

```
.
├── Cargo.toml              # [bin] single crate
└── src/
    ├── domain/
    │   ├── entities/
    │   ├── repositories.rs    # DB traits
    │   ├── ports.rs           # External service traits
    │   └── services.rs
    ├── application/
    │   └── handlers.rs        # Inject repos + ports
    ├── infra/
    │   ├── http/              # Inbound
    │   ├── grpc/
    │   │   ├── server.rs      # Inbound
    │   │   └── clients/       # Outbound
    │   ├── external/          # Outbound
    │   ├── persistence/       # Outbound
    │   └── messaging/
    └── main.rs
```

## Key Patterns

### Entity (Tier 1+)

```rust
pub struct Order {
    id: OrderId,
    items: Vec<OrderItem>,
    status: OrderStatus,
}

impl Order {
    pub fn add_item(&mut self, item: OrderItem) -> Result<(), DomainError> {
        if self.status != OrderStatus::Draft {
            return Err(DomainError::InvalidState);
        }
        self.items.push(item);
        Ok(())
    }
}
```

### Repository Trait (Tier 1+)

```rust
// domain/repositories.rs
#[async_trait]
pub trait OrderRepository {
    async fn find(&self, id: OrderId) -> Result<Option<Order>, RepoError>;
    async fn save(&self, order: &Order) -> Result<(), RepoError>;
}

// infra/persistence/order.rs
impl OrderRepository for SeaOrmOrderRepo {
    async fn find(&self, id: OrderId) -> Result<Option<Order>, RepoError> {
        // sea-orm query, map to domain entity
    }
}
```

### Application Service (Tier 2+)

```rust
pub struct CreateOrderHandler<R: OrderRepository> {
    repo: R,
}

impl<R: OrderRepository> CreateOrderHandler<R> {
    pub async fn handle(&self, cmd: CreateOrder) -> Result<OrderId, AppError> {
        let order = Order::new(cmd.customer_id);
        self.repo.save(&order).await?;
        Ok(order.id)
    }
}
```

### Value Object (Tier 3+)

```rust
// domain/value_objects/money.rs
#[derive(Clone, PartialEq, Eq)]
pub struct Money {
    amount: Decimal,
    currency: Currency,
}

impl Money {
    pub fn new(amount: Decimal, currency: Currency) -> Result<Self, DomainError> {
        if amount < Decimal::ZERO {
            return Err(DomainError::NegativeAmount);
        }
        Ok(Self { amount, currency })
    }

    pub fn add(&self, other: &Money) -> Result<Money, DomainError> {
        if self.currency != other.currency {
            return Err(DomainError::CurrencyMismatch);
        }
        Ok(Money { amount: self.amount + other.amount, currency: self.currency })
    }
}
```

### Aggregate (Tier 3+)

```rust
// domain/aggregates/order/order.rs
pub struct Order {
    id: OrderId,
    items: Vec<OrderItem>,
    status: OrderStatus,
    total: Money,
    events: Vec<OrderEvent>,  // Uncommitted events
}

impl Order {
    pub fn submit(&mut self) -> Result<(), DomainError> {
        if self.items.is_empty() {
            return Err(DomainError::EmptyOrder);
        }
        self.status = OrderStatus::Submitted;
        self.events.push(OrderEvent::Submitted { id: self.id });
        Ok(())
    }

    pub fn take_events(&mut self) -> Vec<OrderEvent> {
        std::mem::take(&mut self.events)
    }
}
```

### Domain Event (Tier 3+)

```rust
pub enum OrderEvent {
    Created { id: OrderId, customer_id: CustomerId },
    Submitted { id: OrderId },
    Shipped { id: OrderId, tracking: TrackingNumber },
}

// application/event_handlers.rs
pub struct OrderEventHandler<N: NotificationService> {
    notifications: N,
}

impl<N: NotificationService> OrderEventHandler<N> {
    pub async fn handle(&self, event: OrderEvent) -> Result<(), AppError> {
        match event {
            OrderEvent::Submitted { id } => {
                self.notifications.send_order_confirmation(id).await?;
            }
            _ => {}
        }
        Ok(())
    }
}
```

### Read Model / Projection (Tier 4+)

```rust
// application/queries/read_models.rs
#[derive(Serialize)]
pub struct OrderSummary {
    pub id: String,
    pub customer_name: String,
    pub item_count: u32,
    pub total: String,
    pub status: String,
}

// infra/projections/order_summary.rs
pub struct OrderSummaryProjector {
    read_db: ReadDatabase,
}

impl OrderSummaryProjector {
    pub async fn apply(&self, event: OrderEvent) -> Result<(), ProjectionError> {
        match event {
            OrderEvent::Created { id, customer_id } => {
                self.read_db.insert_order_summary(id, customer_id).await?;
            }
            OrderEvent::Submitted { id } => {
                self.read_db.update_order_status(id, "submitted").await?;
            }
            _ => {}
        }
        Ok(())
    }
}
```

### Integration Event (Tier 5)

```rust
// integration-events/src/lib.rs
#[derive(Serialize, Deserialize)]
pub enum IntegrationEvent {
    OrderPlaced { order_id: String, items: Vec<ItemDto> },
    InventoryReserved { order_id: String },
    PaymentCompleted { order_id: String, amount: String },
}
```

### Saga Pattern (Tier 5)

```rust
// saga-orchestrator/src/order_saga.rs
pub struct OrderSaga {
    state: SagaState,
    steps: Vec<SagaStep>,
}

pub enum SagaStep {
    ReserveInventory { items: Vec<ItemId> },
    ProcessPayment { amount: Money },
    ShipOrder { address: Address },
}

impl OrderSaga {
    pub async fn execute<C: SagaContext>(&mut self, ctx: &C) -> Result<(), SagaError> {
        for step in &self.steps {
            match step {
                SagaStep::ReserveInventory { items } => {
                    ctx.inventory().reserve(items.clone()).await
                        .map_err(|e| self.compensate(ctx, e))?;
                }
                // ... other steps
            }
        }
        Ok(())
    }

    async fn compensate<C: SagaContext>(&self, ctx: &C, error: impl Error) -> SagaError {
        for completed in self.completed_steps().rev() {
            match completed {
                SagaStep::ReserveInventory { items } => {
                    let _ = ctx.inventory().release(items.clone()).await;
                }
                // ... compensate other steps
            }
        }
        SagaError::Compensated(error.to_string())
    }
}
```

### Outbox Pattern (Tier 4+)

```rust
// infra/persistence/outbox.rs
#[derive(Model)]
pub struct OutboxMessage {
    pub id: Uuid,
    pub aggregate_type: String,
    pub aggregate_id: String,
    pub event_type: String,
    pub payload: Json,
    pub created_at: DateTime<Utc>,
    pub processed_at: Option<DateTime<Utc>>,
}

pub async fn save_with_events<T: Aggregate>(
    txn: &DatabaseTransaction,
    aggregate: &T,
    events: Vec<DomainEvent>,
) -> Result<(), DbError> {
    aggregate.save(txn).await?;
    for event in events {
        OutboxMessage::from_event(event).save(txn).await?;
    }
    Ok(())
}
```

## Evolution Path

```
Tier 1 → 2: Add application layer, split commands/queries
Tier 2 → 3: Introduce aggregates, value objects, domain events
Tier 3 → 4: Separate read/write stores, add projections
Tier 4 → 5: Split bounded contexts, add integration events, sagas
```

Upgrade signals:

| Signal | Upgrade To |
|--------|------------|
| Use cases scattered across handlers | Tier 2 |
| Hard to test without infrastructure | Tier 2 |
| Complex invariants spanning entities | Tier 3 |
| Need to react to domain changes | Tier 3 |
| Read queries slow down writes | Tier 4 |
| Need complete audit history | Tier 4 |
| Teams stepping on each other | Tier 5 |
| Need independent deployments | Tier 5 |
| Cross-service transactions required | Tier 5 |

## Dependency Injection

| Tier | DI Approach |
|------|-------------|
| Tier 1-2 | Generics + trait bounds |
| Tier 3-4 | Generics + manual wiring |
| Tier 5 | Per-context composition roots |

### Tier 1-2: Simple Generic

```rust
// Concrete (sufficient for most cases)
pub struct OrderService { repo: SeaOrmOrderRepo }

// Generic (when testing needed)
pub struct OrderService<R: OrderRepository> { repo: R }
```

### Tier 3+: Standard

```rust
// main.rs - manual wiring (recommended)
fn main() {
    let db = Database::connect(&config.db_url).await?;
    let order_repo = SeaOrmOrderRepo::new(db.clone());
    let payment = StripeGateway::new(&config.stripe_key);

    let handler = Arc::new(CreateOrderHandler::new(order_repo, payment));

    let app = Router::new()
        .route("/orders", post(create_order))
        .with_state(AppState { handler });
}
```

```rust
// application/handlers.rs
pub struct CreateOrderHandler<R: OrderRepository, P: PaymentGateway> {
    repo: R,
    payment: P,
}

impl<R: OrderRepository, P: PaymentGateway> CreateOrderHandler<R, P> {
    pub fn new(repo: R, payment: P) -> Self {
        Self { repo, payment }
    }
}
```

### Testing

```rust
#[cfg(test)]
mod tests {
    #[tokio::test]
    async fn test_create_order() {
        let handler = CreateOrderHandler::new(MockRepo::new(), MockPayment::new());
        assert!(handler.handle(cmd).await.is_ok());
    }
}
```

### DI Anti-patterns

- Don't create traits for every dependency
- Don't use runtime DI containers (prefer compile-time)
- Don't over-abstract internal services

## Anti-patterns

| Anti-pattern | Applies To |
|--------------|------------|
| Business logic in HTTP handlers | All tiers |
| ORM entities leaking into domain | All tiers |
| Application layer without clear use cases | Tier 2+ |
| Aggregates for simple CRUD | Tier 3+ |
| Domain events without consumers | Tier 3+ |
| CQRS without read/write disparity | Tier 4 |
| Premature bounded context splits | Tier 5 |
| Sagas for local transactions | Tier 5 |
