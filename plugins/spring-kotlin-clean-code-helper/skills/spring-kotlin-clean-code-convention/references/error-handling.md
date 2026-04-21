# Error Handling

Exceptions make program flow explicit. Generic exceptions leave the caller with no idea what went wrong — always throw something specific.

## Custom Exceptions

```kotlin
// ❌ Generic — caller can't tell what happened
throw RuntimeException("User not found")
throw IllegalArgumentException("Invalid order status")

// ✅ Domain-specific — intent is clear
class UserNotFoundException(userId: Long) :
    BusinessException("User not found: $userId", ErrorCode.USER_NOT_FOUND)

class InvalidOrderStatusException(current: OrderStatus, expected: OrderStatus) :
    BusinessException(
        "Order status transition not allowed: $current → $expected",
        ErrorCode.INVALID_ORDER_STATUS
    )
```

## Exception Hierarchy

```kotlin
sealed class BusinessException(
    message: String,
    val errorCode: ErrorCode,
    cause: Throwable? = null
) : RuntimeException(message, cause)

class NotFoundException(message: String, errorCode: ErrorCode) : BusinessException(message, errorCode)
class ValidationException(message: String, errorCode: ErrorCode) : BusinessException(message, errorCode)
class ConflictException(message: String, errorCode: ErrorCode) : BusinessException(message, errorCode)
```

## Sealed Class for Outcomes (Internal Logic)

When a function can fail in multiple expected ways, model it as a type rather than throwing:

```kotlin
sealed class OrderResult {
    data class Success(val order: Order) : OrderResult()
    data class InsufficientStock(val itemId: Long, val available: Int) : OrderResult()
    data class UserNotEligible(val reason: String) : OrderResult()
}
```

## Null Returns

Avoid returning null to signal "not found". Use `find*` naming when null is intentionally possible.

```kotlin
// ❌ Caller must remember to null-check
fun getUser(id: Long): User? = userRepository.findById(id).orElse(null)

// ✅ Throws if not found — no silent null propagation
fun getUser(id: Long): User = userRepository.findById(id)
    .orElseThrow { UserNotFoundException(id) }

// find* convention — null return is expected and documented by the name
fun findUser(id: Long): User? = userRepository.findById(id).orElse(null)
```

## Centralized Exception Handling

```kotlin
@RestControllerAdvice
class GlobalExceptionHandler {
    @ExceptionHandler(NotFoundException::class)
    fun handleNotFound(e: NotFoundException): ResponseEntity<ErrorResponse> =
        ResponseEntity.status(HttpStatus.NOT_FOUND).body(ErrorResponse(e.errorCode, e.message))

    @ExceptionHandler(ValidationException::class)
    fun handleValidation(e: ValidationException): ResponseEntity<ErrorResponse> =
        ResponseEntity.status(HttpStatus.BAD_REQUEST).body(ErrorResponse(e.errorCode, e.message))
}
```
