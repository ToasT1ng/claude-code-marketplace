# SOLID Principles

## S — Single Responsibility

A class should have only one reason to change. If you find yourself describing a class with "and", it probably does too much.

```kotlin
// ❌ RegisterUserService doing too much
class UserService {
    fun register(request: RegisterRequest) { ... }
    fun sendWelcomeEmail(user: User) { ... }     // belongs in UserEmailService
    fun calculateUserScore(user: User) { ... }   // belongs in UserScoringService
}

// ✅ Separated by responsibility
class RegisterUserService(
    private val userEmailService: UserEmailService,
    private val userScoringService: UserScoringService
) {
    fun register(command: RegisterUserCommand): User { ... }
}
```

## O — Open/Closed

Open for extension, closed for modification. Adding a new behavior should not require editing existing code.

```kotlin
// ❌ Every new grade requires modifying this function
fun calculateDiscount(user: User, price: Money): Money = when (user.grade) {
    UserGrade.BRONZE -> price * 0.0
    UserGrade.SILVER -> price * 0.05
    UserGrade.GOLD -> price * 0.10
}

// ✅ New grades = new class, no edits to existing code
interface DiscountPolicy {
    fun calculate(price: Money): Money
}

class GoldDiscountPolicy : DiscountPolicy {
    override fun calculate(price: Money) = price * 0.10
}
```

## L — Liskov Substitution

Subclasses must be fully substitutable for their parent without breaking the program. In practice: prefer interfaces and composition over inheritance.

## I — Interface Segregation

Don't force a class to depend on methods it doesn't use. Split large interfaces by purpose.

```kotlin
// ❌ Read-only consumer forced to know about mutations
interface UserRepository : JpaRepository<User, Long>

// ✅ Separate read/write concerns (useful in CQRS)
interface UserReadRepository {
    fun findById(id: Long): User?
    fun findActiveUsers(): List<User>
}
```

## D — Dependency Inversion

High-level modules must not depend on low-level implementations. Always use **constructor injection** — never field injection.

```kotlin
// ❌ Field injection — hard to test, dependencies are hidden
@Service
class OrderService {
    @Autowired
    private lateinit var orderRepository: OrderRepository
}

// ✅ Constructor injection — dependencies are explicit and testable
@Service
class CreateOrderService(
    private val orderRepository: OrderRepository,
    private val paymentClient: PaymentClient
)
```
