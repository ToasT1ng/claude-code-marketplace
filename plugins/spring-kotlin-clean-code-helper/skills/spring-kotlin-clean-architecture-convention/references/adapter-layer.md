# Adapter Layer

Adapters are the bridge between the outside world and the application. They translate external representations (HTTP requests, JPA rows, API responses) into domain/application types — and back. Adapters know about ports; ports know nothing about adapters.

## Inbound Adapters (`adapter/in/`)

Driving adapters receive external input and call input ports.

### Web (Controller)

Naming follows `{Verb}{Domain}Controller` (same as clean-code-convention).

```kotlin
// adapter/in/web/CreateOrderController.kt
@RestController
@RequestMapping("/api/v1/orders")
class CreateOrderController(
    private val createOrderUseCase: CreateOrderUseCase  // depends on port, not service
) {
    @PostMapping
    fun createOrder(
        @RequestBody @Valid request: CreateOrderRequest,
        @AuthenticationPrincipal principal: UserPrincipal
    ): ResponseEntity<CreateOrderResponse> {
        val command = request.toCommand(UserId(principal.userId))
        val result = createOrderUseCase.createOrder(command)
        return ResponseEntity.status(HttpStatus.CREATED).body(result.toResponse())
    }
}
```

Controllers depend on **use case interfaces** (`CreateOrderUseCase`), never on concrete services. This keeps the controller decoupled from the implementation.

### Controller DTOs

Controller DTOs (`{Domain}Request`, `{Domain}Response`) belong to the adapter layer, not the application layer.

```kotlin
// adapter/in/web/dto/
data class CreateOrderRequest(
    @field:NotEmpty val items: List<OrderItemRequest>
)

data class OrderItemRequest(val productId: Long, val quantity: Int)

data class CreateOrderResponse(val orderId: Long, val status: String, val totalPrice: BigDecimal)

// Conversion: adapter DTO → application command (adapter's responsibility)
fun CreateOrderRequest.toCommand(userId: UserId) = CreateOrderCommand(
    userId = userId,
    items = items.map { OrderItemCommand(it.productId, it.quantity) }
)

fun CreateOrderResult.toResponse() = CreateOrderResponse(
    orderId = order.id.value,
    status = order.status.name,
    totalPrice = order.totalPrice.amount
)
```

## Outbound Adapters (`adapter/out/`)

Driven adapters implement output ports to provide infrastructure capabilities to the application.

### Persistence (`adapter/out/infra/`)

```kotlin
// adapter/out/infra/OrderPersistenceAdapter.kt
@Component
class OrderPersistenceAdapter(
    private val orderJpaRepository: OrderJpaRepository
) : OrderPort {

    override fun save(order: Order): Order =
        orderJpaRepository.save(order.toJpaEntity()).toDomain()

    override fun findById(orderId: OrderId): Order? =
        orderJpaRepository.findById(orderId.value).orElse(null)?.toDomain()

    override fun findAllByUserId(userId: UserId): List<Order> =
        orderJpaRepository.findByUserId(userId.value).map { it.toDomain() }
}

// JPA entity lives here — not in domain
@Entity
@Table(name = "orders")
class OrderJpaEntity(
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    val id: Long? = null,
    val userId: Long,
    val status: String,
    val totalAmount: BigDecimal,
    val currency: String
)

// Conversion functions (adapter's responsibility)
fun Order.toJpaEntity() = OrderJpaEntity(
    id = id.value.takeIf { it != 0L },
    userId = userId.value,
    status = status.name,
    totalAmount = totalPrice.amount,
    currency = totalPrice.currency.currencyCode
)

fun OrderJpaEntity.toDomain() = Order(
    id = OrderId(id!!),
    userId = UserId(userId),
    items = emptyList(), // load separately if needed
    status = OrderStatus.valueOf(status),
    totalPrice = Money(totalAmount, Currency.getInstance(currency))
)
```

### External API Client (`adapter/out/api/`)

```kotlin
// adapter/out/api/PaymentApiAdapter.kt
@Component
class PaymentApiAdapter(
    private val paymentFeignClient: PaymentFeignClient
) : PaymentPort {

    override fun requestPayment(order: Order): PaymentResult {
        val response = paymentFeignClient.pay(order.toPaymentRequest())
        return response.toDomain()
    }
}
```

## Key Rules

- Adapters implement ports — they are **never** called directly by other adapters
- `adapter/in` and `adapter/out` must not import each other
- All domain↔adapter type conversion happens in the adapter, not the domain
- JPA entities, Feign request/response types, and HTTP DTOs all live in the adapter layer
