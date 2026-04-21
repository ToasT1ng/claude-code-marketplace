# Kotlin Idioms

Non-idiomatic Kotlin obscures intent. Java-style Kotlin is a code smell — use the language's features.

## val vs var

Prefer `val`. Use `var` only when mutation is truly unavoidable (e.g., loop counters, builders).

```kotlin
val userId = request.userId   // ✅
var count = 0                 // only when you must mutate
```

## Immutable Domain Models

```kotlin
data class Money(val amount: BigDecimal, val currency: Currency) {
    operator fun plus(other: Money) = Money(amount + other.amount, currency)
    operator fun times(rate: Double) = Money(amount * rate.toBigDecimal(), currency)
}
```

## Scope Functions

Each scope function has a clear purpose — don't use them interchangeably.

```kotlin
// also: side effect (logging, auditing) without changing the value
val user = userRepository.findById(id)?.also {
    log.info("User found: ${it.id}")
} ?: throw UserNotFoundException(id)

// let: transform after null check
val userName = user?.let { "${it.firstName} ${it.lastName}" }

// apply: object initialization / builder pattern
val order = Order().apply {
    this.user = currentUser
    this.status = OrderStatus.PENDING
}

// run: compute a result using the receiver
val summary = order.run { "${user.name}: $totalPrice" }
```

## Extension Functions

Use extension functions to add domain expressiveness without polluting the class itself.

```kotlin
fun Order.isModifiable(): Boolean = status in listOf(OrderStatus.PENDING, OrderStatus.CONFIRMED)
fun Money.isZero(): Boolean = amount == BigDecimal.ZERO
```

## Exhaustive when

Never use `else` on sealed classes or enums — exhaustive `when` gives compile-time safety.

```kotlin
// ✅ If a new status is added, this won't compile until handled
fun describe(status: OrderStatus): String = when (status) {
    OrderStatus.PENDING -> "Awaiting payment"
    OrderStatus.CONFIRMED -> "Order confirmed"
    OrderStatus.SHIPPED -> "Shipped"
    OrderStatus.DELIVERED -> "Delivered"
    OrderStatus.CANCELLED -> "Cancelled"
}
```

## Null Safety

Don't fight the type system — use it.

```kotlin
// ❌ Force-unwrapping defeats null safety
val name = user!!.name

// ✅ Handle null explicitly
val name = user?.name ?: "Unknown"
```
