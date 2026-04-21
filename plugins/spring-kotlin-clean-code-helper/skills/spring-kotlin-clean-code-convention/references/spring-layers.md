# Spring Layer Rules

## Class Naming: `{Verb}{Domain}{Layer}`

Controller and Service classes are split **per use case**. Class names follow `{Verb}{Domain}{Layer}` in PascalCase.
Repository is grouped by domain (not by use case) since it's the persistence boundary.

```
GetOrderController        CreateOrderController        CancelOrderController
GetOrderService           CreateOrderService           CancelOrderService
OrderRepository           (one per domain aggregate)
```

## Layer Communication via DTOs Only

Each layer knows only its own DTOs. Never pass another layer's type directly across boundaries.
The **receiving layer** is responsible for conversion.

```
HTTP Request
    ↓  {Domain}Request
Controller
    ↓  {Domain}Command
Service
    ↓  {Domain}Query  /  JpaEntity
Repository
```

## DTO Naming Rules

All DTOs follow `{Verb}{Domain}{Suffix}` in PascalCase.

| Layer | Input | Output |
|-------|-------|--------|
| Controller | `{Domain}Request` | `{Domain}Response` |
| Service | `{Domain}Command` | `{Domain}Result` or Domain object |
| Repository | `{Domain}Query` | Domain object or `{Domain}JpaEntity` |

**Example — Order domain**
```kotlin
// Controller
data class CreateOrderRequest(val userId: Long, val items: List<OrderItemRequest>)
data class CreateOrderResponse(val orderId: Long, val status: String, val totalPrice: BigDecimal)

// Service
data class CreateOrderCommand(val userId: Long, val items: List<OrderItemCommand>)
data class CreateOrderResult(val order: Order)

// Repository
data class FindOrderQuery(val userId: Long, val status: OrderStatus?, val fromDate: LocalDate?)
// Returns: Order (domain) or OrderJpaEntity
```

## Layer Examples

**Controller** — HTTP translation only, no business logic
```kotlin
@RestController
@RequestMapping("/api/v1/orders")
class CreateOrderController(
    private val createOrderService: CreateOrderService  // inject concrete service in layered arch
) {
    @PostMapping
    fun createOrder(
        @RequestBody @Valid request: CreateOrderRequest,
        @AuthenticationPrincipal principal: UserPrincipal
    ): ResponseEntity<CreateOrderResponse> {
        val command = request.toCommand(principal.userId)
        val result = createOrderService.create(command)
        return ResponseEntity.status(HttpStatus.CREATED).body(result.toResponse())
    }
}
```

> **Hexagonal Architecture**: If following hexagonal architecture, inject the use case interface instead of the concrete service — `private val createOrderUseCase: CreateOrderUseCase`. See `spring-kotlin-clean-architecture-convention` skill for details.

**Service** — business logic and transaction boundary
```kotlin
@Service
@Transactional(readOnly = true)
class CreateOrderService(
    private val orderRepository: OrderRepository,
    private val stockService: StockService
) {
    @Transactional
    fun create(command: CreateOrderCommand): CreateOrderResult {
        stockService.reserve(command.items)
        val saved = orderRepository.save(command.toDomain())
        return CreateOrderResult(saved)
    }
}
```

**Repository** — data access only
```kotlin
interface OrderRepository : JpaRepository<OrderJpaEntity, Long> {
    fun findByUserId(userId: Long): List<OrderJpaEntity>

    @Query("SELECT o FROM OrderJpaEntity o WHERE o.userId = :#{#query.userId} AND o.status != 'CANCELLED'")
    fun findActiveOrders(@Param("query") query: FindOrderQuery): List<OrderJpaEntity>
}
```

## DTO Conversion via Extension Functions

```kotlin
// ❌ Service must not know about Controller DTOs
class CreateOrderService {
    fun create(request: CreateOrderRequest): CreateOrderResponse  // forbidden
}

// ✅ Each DTO owns its conversion logic as an extension function
fun CreateOrderRequest.toCommand(userId: Long) = CreateOrderCommand(userId, items.map { it.toCommand() })
fun CreateOrderResult.toResponse() = CreateOrderResponse(order.id, order.status.name, order.totalPrice)
fun CreateOrderCommand.toDomain() = Order(userId = userId, items = items.map { it.toDomain() })
```
