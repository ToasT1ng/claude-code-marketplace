# Application Layer

The application layer orchestrates domain logic. It defines what the system *can do* (input ports) and what it *needs* from the outside world (output ports). It must not know how adapters implement those needs.

## Input Ports (Use Case Interfaces)

Input ports live in `domain/port/in/` — they are part of the domain boundary, not the application layer.
The application layer *implements* them; the domain layer *owns* them.
This way the domain defines what operations are possible, and adapters depend on the domain — not on application internals.

Naming: `{Verb}{Domain}UseCase`

```kotlin
// domain/port/in/CreateOrderUseCase.kt
interface CreateOrderUseCase {
    fun createOrder(command: CreateOrderCommand): CreateOrderResult
}

// domain/port/in/CancelOrderUseCase.kt
interface CancelOrderUseCase {
    fun cancelOrder(command: CancelOrderCommand): CancelOrderResult
}
```

## Output Ports (Repository / Client Interfaces)

Output ports live in `domain/port/out/`. They define what the application layer needs from the infrastructure — without knowing how it's done.

Naming: `{Domain}Port`

```kotlin
// domain/port/out/OrderPort.kt
interface OrderPort {
    fun save(order: Order): Order
    fun findById(orderId: OrderId): Order?
    fun findAllByUserId(userId: UserId): List<Order>
}

// domain/port/out/StockPort.kt
interface StockPort {
    fun reserve(items: List<OrderItem>): Boolean
}

// domain/port/out/PaymentPort.kt
interface PaymentPort {
    fun requestPayment(order: Order): PaymentResult
}
```

## Application Services (Use Case Implementations)

Application services live in `application/service/`. They implement input ports and depend on output ports — never on concrete adapters.

Naming: `{Verb}{Domain}Service`

```kotlin
// application/service/CreateOrderService.kt
@Service
@Transactional
class CreateOrderService(
    private val orderPort: OrderPort,
    private val stockPort: StockPort
) : CreateOrderUseCase {

    override fun createOrder(command: CreateOrderCommand): CreateOrderResult {
        stockPort.reserve(command.items)
        val order = Order.create(command.userId, command.items)
        val saved = orderPort.save(order)
        return CreateOrderResult(saved)
    }
}
```

## Application-Layer DTOs

Use case commands and results carry data across the boundary between adapter and application. They belong to the application layer and use only domain types or primitives.

```kotlin
// application-layer DTOs
data class CreateOrderCommand(
    val userId: UserId,
    val items: List<OrderItemCommand>
)

data class OrderItemCommand(val productId: Long, val quantity: Int)

data class CreateOrderResult(val order: Order)
```

## What Does NOT Belong in the Application Layer

- HTTP types (`HttpServletRequest`, `ResponseEntity`)
- JPA entities (`OrderJpaEntity`)
- Adapter-specific details (e.g., Feign client configs)

```kotlin
// ❌ Application service importing JPA entity
class CreateOrderService(
    private val orderJpaRepository: OrderJpaRepository  // wrong — use port
)

// ❌ Application service returning HTTP response
fun createOrder(command: CreateOrderCommand): ResponseEntity<OrderResponse>  // wrong
```
