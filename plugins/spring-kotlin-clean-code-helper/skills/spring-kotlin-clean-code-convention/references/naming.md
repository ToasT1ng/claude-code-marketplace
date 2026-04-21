# Naming Conventions

Names must reveal intent. Reading a name should immediately tell you what it does — no mental translation required.

## Classes / Interfaces

- Noun or noun phrase: `UserService`, `OrderRepository`, `PaymentProcessor`
- No `I` prefix on interfaces: `IUserService` → `UserService`
- Distinguish implementations by role, not by suffix: `UserServiceImpl` → `DefaultUserService`, `CachedUserService`

## Functions

- Verb or verb phrase: `findActiveUsers()`, `calculateTotalPrice()`, `validatePayment()`
- Boolean returns start with `is`, `has`, `can`, `should`: `isExpired()`, `hasPermission()`
- Make side effects visible in the name: `saveAndNotify()`, `deleteWithAudit()`

## Variables / Properties

- No abbreviations (except universally understood ones): `usr` → `user`, `cnt` → `count`
- Replace magic numbers with named constants: `if (age > 18)` → `if (age > LEGAL_AGE)`
- Collections in plural: `userList` → `users`, `orderIds`

## Packages

- Lowercase, singular: `user`, `order`, `payment`
- Reflect the layer: `com.example.user.service`, `com.example.order.repository`
