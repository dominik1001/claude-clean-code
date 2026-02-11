---
name: unit-testing
description: Best practices for writing unit tests in TypeScript/JavaScript. Use when writing, reviewing, or refactoring unit tests, when setting up test structure and naming conventions, when working with mocks/stubs/fakes, or when improving test quality (trustworthiness, maintainability, readability).
---

# Unit Testing Best Practices

Guidelines for writing high-quality unit tests based on Roy Osherove's "The Art of Unit Testing".

## The Three Pillars

Every test should be:

1. **Trustworthy** — Fails only for real bugs, no flakiness, no logic in tests
2. **Maintainable** — Easy to update when requirements change, isolated, not brittle
3. **Readable** — Clear naming, obvious intent, anyone can understand quickly

## Core Practices

### 1. Behavior-Focused Test Names

Describe *what the system does*, not which method is called. Use `describe`/`it` blocks that read like specifications.

```typescript
// ✅ Good
describe("ShoppingCart", () => {
  it("returns zero when the cart is empty", () => {
    expect(new ShoppingCart().calculateTotal()).toBe(0);
  });
});

// ❌ Avoid cryptic or method-focused names
test("test1", () => { /* ... */ });
test("calculateTotal_emptyCart_zero", () => { /* ... */ });
```

### 2. No Logic in Tests

Tests should have no conditionals, loops, or dynamic calculations. Use hardcoded expected values.

```typescript
// ✅ Good — hardcoded expectation
it("calculates 10% discount on $100", () => {
  expect(calculateDiscount(100)).toBe(10);
});

// ❌ Bad — logic in test
it("calculates discount", () => {
  const price = 100;
  expect(calculateDiscount(price)).toBe(price * 0.1); // Logic!
});
```

### 3. AAA Pattern (Arrange-Act-Assert)

Structure every test in three clear sections.

```typescript
it("saves user to database", async () => {
  // Arrange
  const repo = new FakeUserRepository();
  const service = new UserService(repo);

  // Act
  await service.createUser({ name: "Alice" });

  // Assert
  expect(repo.savedUsers).toContainEqual({ name: "Alice" });
});
```

### 4. One Concept Per Test

Each test verifies one logical concept. Split tests that verify multiple behaviors.

```typescript
// ✅ Good — separate tests for separate behaviors
it("sets the user name correctly", () => {
  expect(createUser("Bob").name).toBe("Bob");
});

it("sends welcome email on creation", () => {
  const emailService = mockEmailService();
  createUser("Bob", { emailService });
  expect(emailService.sendWelcome).toHaveBeenCalled();
});
```

### 5. Stubs vs Mocks

- **Stub**: Returns canned data (for inputs). Preferred.
- **Mock**: Verifies interactions (for outputs). Use sparingly.

```typescript
// Stub — controls input
const fakeTimeProvider = { now: () => new Date("2024-01-15") };

// Mock — verifies interaction
const warehouseMock = { notify: vi.fn() };
orderService.placeOrder(order);
expect(warehouseMock.notify).toHaveBeenCalledWith("123", 2);
```

### 6. Test Isolation with Factories

Avoid shared state via `beforeEach`. Each test creates its own world using factories/builders.

```typescript
// ✅ Good — factory per test
function createCart(...items: Item[]): ShoppingCart {
  const cart = new ShoppingCart();
  items.forEach(item => cart.add(item));
  return cart;
}

it("charges correct amount for single item", () => {
  const cart = createCart({ id: "1", price: 50 });
  expect(cart.checkout()).toBe(50);
});
```

### 7. Separate Unit and Integration Tests

Keep fast, reliable unit tests in a "safe green zone". Integration tests live separately.

```
tests/
├── unit/           # Fast, no external deps
└── integration/    # Requires DB, network, etc.
```

## Quick Reference

For detailed examples and patterns, see [references/examples.md](references/examples.md).
