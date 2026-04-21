# Package Structure & Dependency Rules

## Directory Layout

```
com.example.{domain-context}/
├── adapter/
│   ├── in/
│   │   └── web/                  ← HTTP controllers
│   └── out/
│       ├── infra/                ← JPA repositories, persistence adapters
│       └── api/                  ← External API clients
├── application/
│   ├── service/                  ← Use case implementations
│   └── util/                     ← Application-level utilities
└── domain/
    ├── model/                    ← Entities, Value Objects, Domain Services
    └── port/
        ├── in/                   ← Input port interfaces (use case contracts)
        └── out/                  ← Output port interfaces (repo, client contracts)
```

## Dependency Rule

```
adapter/in  ──▶  application  ──▶  domain
adapter/out ──▶  application  ──▶  domain
                 application  ──▶  domain
```

- `domain` has zero dependencies on any other layer — no Spring, no JPA, no adapter types
- `application` depends only on `domain`; it knows ports but not adapters
- `adapter` depends on `application` ports and `domain` models; never the reverse

## What Each Layer May Import

| Layer | May import | Must NOT import |
|-------|-----------|-----------------|
| `domain` | Nothing outside domain | `application`, `adapter`, Spring, JPA |
| `application` | `domain` only | `adapter`, Spring MVC, JPA entities |
| `adapter/in` | `application` ports, `domain` models | `adapter/out` internals |
| `adapter/out` | `application` ports, `domain` models | `adapter/in` internals |

## Violation Examples

```kotlin
// ❌ Application layer importing a JPA entity (adapter concern leaked in)
import com.example.order.adapter.out.infra.OrderJpaEntity

// ❌ Domain importing a Spring annotation
import org.springframework.stereotype.Component
@Component  // forbidden in domain
class Order(...)

// ❌ Controller calling repository directly (skipping application layer)
@RestController
class CreateOrderController(
    private val orderRepository: OrderJpaRepository  // wrong — use port/service
)
```

## Multiple Bounded Contexts

Each bounded context gets its own top-level package. They communicate through domain events or application-layer interfaces — never by directly importing each other's internals.

```
com.example.order/        ← Order context
com.example.payment/      ← Payment context
com.example.notification/ ← Notification context
```
