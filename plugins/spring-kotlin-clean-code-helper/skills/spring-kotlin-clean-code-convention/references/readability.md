# Readability — Code That Reads Like a Newspaper

A good article flows from headline → summary → details. Code should read the same way — top to bottom, naturally, without jumping around.

## Functions Do One Thing

```kotlin
// ❌ Function doing too many things at different abstraction levels
fun processOrder(order: Order) {
    validateOrder(order)
    val price = order.items.sumOf { it.price * it.quantity }
    val discount = if (order.user.isPremium) price * 0.1 else 0.0
    orderRepository.save(order.copy(totalPrice = price - discount))
    emailService.sendConfirmation(order.user.email, order)
    slackService.notify("#orders", "New order: ${order.id}")
}

// ✅ One level of abstraction per function
fun processOrder(order: Order) {
    val validatedOrder = validate(order)
    val pricedOrder = applyPricing(validatedOrder)
    save(pricedOrder)
    notifyCompletion(pricedOrder)
}
```

## Function Size

Keep functions under ~20 lines. If it doesn't fit on one screen without scrolling, consider splitting.

## Consistent Abstraction Level

Don't mix high-level logic (`validateOrder`) with low-level implementation (`order.items.sumOf { it.price * it.quantity }`) inside the same function. Each function should sit at one level.

## File Structure: Public First, Private Below

```kotlin
class OrderService(...) {
    fun createOrder(command: CreateOrderCommand): Order { ... }   // public
    fun cancelOrder(orderId: Long): Order { ... }                 // public

    private fun validateStock(items: List<OrderItem>) { ... }
    private fun calculateTotal(items: List<OrderItem>): Money { ... }
}
```

## Comments: Why, Not What

If the code is clear, no comment is needed. When you do comment, explain the business reason — not what the code already says.

```kotlin
// ❌ Restates the code
// If user age is greater than 18, treat as adult
if (user.age > LEGAL_AGE) { ... }

// ✅ Explains the business constraint
// Korean law requires parental consent for users under 14 (ICNA Article 31)
if (user.age < MIN_AGE_WITHOUT_CONSENT) { ... }
```
