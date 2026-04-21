# Domain Layer

The domain layer is the heart of the application. It must be completely independent — no Spring, no JPA, no framework leakage. If you can't unit-test a domain class without loading a Spring context, something is wrong.

## Entities

An entity has identity that persists over time. Use `data class` for immutability, and model behavior as methods rather than exposing raw state.

```kotlin
// ✅ Pure domain entity — no annotations, no framework dependency
data class Order(
    val id: OrderId,
    val userId: UserId,
    val items: List<OrderItem>,
    val status: OrderStatus,
    val totalPrice: Money
) {
    fun confirm(): Order {
        check(status == OrderStatus.PENDING) { "Only PENDING orders can be confirmed" }
        return copy(status = OrderStatus.CONFIRMED)
    }

    fun cancel(): Order {
        check(status.isCancellable()) { "Order in $status cannot be cancelled" }
        return copy(status = OrderStatus.CANCELLED)
    }
}
```

## Value Objects

A value object has no identity — equality is defined by its values. Use `data class` and make it immutable.

```kotlin
data class Money(val amount: BigDecimal, val currency: Currency) {
    init {
        require(amount >= BigDecimal.ZERO) { "Money amount must be non-negative" }
    }

    operator fun plus(other: Money): Money {
        require(currency == other.currency) { "Cannot add different currencies" }
        return copy(amount = amount + other.amount)
    }

    operator fun times(quantity: Int) = copy(amount = amount * quantity.toBigDecimal())
}

data class OrderId(val value: Long)
data class UserId(val value: Long)
```

Wrapping primitive IDs in value objects (`OrderId`, `UserId`) prevents accidental mix-ups at compile time.

## Domain Services

When a business operation doesn't naturally belong to a single entity, put it in a domain service. Domain services are plain classes — no `@Service` annotation.

```kotlin
class OrderPricingService {
    fun calculateTotal(items: List<OrderItem>, discountPolicy: DiscountPolicy): Money {
        val subtotal = items.sumOf { it.price * it.quantity }
        return discountPolicy.apply(subtotal)
    }
}
```

## What Does NOT Belong in the Domain

- Spring annotations (`@Component`, `@Service`, `@Transactional`)
- JPA annotations (`@Entity`, `@Column`, `@Id`)
- Database or HTTP types
- Anything that requires an application context to instantiate

```kotlin
// ❌ JPA leaking into domain
@Entity
@Table(name = "orders")
data class Order(
    @Id @GeneratedValue
    val id: Long? = null,
    ...
)

// ✅ JPA entity lives in adapter/out/infra — separate from domain
@Entity
@Table(name = "orders")
class OrderJpaEntity(
    @Id @GeneratedValue
    val id: Long? = null,
    val userId: Long,
    val status: String,
    ...
)
```
