# Unit Testing Examples

Detailed examples for each best practice.

## Test Naming Patterns

### BDD-Style with describe/it

```typescript
describe("UserService", () => {
  describe("createUser", () => {
    it("persists the user to the database", async () => { /* ... */ });
    it("sends a welcome email", async () => { /* ... */ });
    it("throws ValidationError when email is invalid", async () => { /* ... */ });
  });

  describe("deleteUser", () => {
    it("removes the user from the database", async () => { /* ... */ });
    it("cancels all active subscriptions", async () => { /* ... */ });
  });
});
```

### Nested Context with Given/When

```typescript
describe("PricingCalculator", () => {
  describe("given a preferred customer", () => {
    it("applies 10% discount", () => { /* ... */ });
    it("waives shipping on orders over $50", () => { /* ... */ });
  });

  describe("given a regular customer", () => {
    it("applies no discount", () => { /* ... */ });
    it("charges standard shipping", () => { /* ... */ });
  });
});
```

## Factory/Builder Patterns

### Simple Factory Function

```typescript
function createUser(overrides: Partial<User> = {}): User {
  return {
    id: "user-123",
    name: "Test User",
    email: "test@example.com",
    createdAt: new Date("2024-01-01"),
    ...overrides,
  };
}

// Usage
it("displays user name", () => {
  const user = createUser({ name: "Alice" });
  expect(formatGreeting(user)).toBe("Hello, Alice!");
});
```

### Builder Pattern for Complex Objects

```typescript
class OrderBuilder {
  private order: Partial<Order> = {
    items: [],
    status: "pending",
  };

  withItem(product: string, qty: number, price: number): this {
    this.order.items!.push({ product, qty, price });
    return this;
  }

  withStatus(status: OrderStatus): this {
    this.order.status = status;
    return this;
  }

  forCustomer(customerId: string): this {
    this.order.customerId = customerId;
    return this;
  }

  build(): Order {
    return this.order as Order;
  }
}

// Usage
it("calculates total with multiple items", () => {
  const order = new OrderBuilder()
    .withItem("Widget", 2, 10)
    .withItem("Gadget", 1, 25)
    .build();

  expect(calculateTotal(order)).toBe(45);
});
```

## Stub and Mock Examples

### Stubbing External Services

```typescript
// Stub for time-dependent code
const fakeTimer: TimeProvider = {
  now: () => new Date("2024-06-15T10:00:00Z"),
};

it("marks subscription as expired when past end date", () => {
  const subscription = new Subscription(
    new Date("2024-06-01"),
    fakeTimer
  );
  expect(subscription.isExpired()).toBe(true);
});

// Stub for API responses
const fakeApiClient: ApiClient = {
  fetchUser: async (id) => ({ id, name: "Stubbed User", email: "stub@test.com" }),
  fetchOrders: async () => [],
};
```

### Mocking with Vitest/Jest

```typescript
import { vi, expect, it, describe, beforeEach } from "vitest";

describe("NotificationService", () => {
  it("sends email when order ships", async () => {
    const emailSender = { send: vi.fn().mockResolvedValue(true) };
    const service = new NotificationService(emailSender);

    await service.notifyShipment({ orderId: "123", email: "user@test.com" });

    expect(emailSender.send).toHaveBeenCalledWith(
      "user@test.com",
      expect.stringContaining("shipped")
    );
  });

  it("logs error when email fails", async () => {
    const emailSender = { send: vi.fn().mockRejectedValue(new Error("SMTP down")) };
    const logger = { error: vi.fn() };
    const service = new NotificationService(emailSender, logger);

    await service.notifyShipment({ orderId: "123", email: "user@test.com" });

    expect(logger.error).toHaveBeenCalledWith(expect.stringContaining("SMTP"));
  });
});
```

### Partial Mocks (Spies)

```typescript
it("calls analytics when user signs up", () => {
  const analytics = { track: vi.fn() };
  const realUserService = new UserService();
  
  // Spy on specific method while keeping real implementation
  vi.spyOn(realUserService, "validate").mockReturnValue(true);

  realUserService.signUp({ email: "new@test.com" }, analytics);

  expect(analytics.track).toHaveBeenCalledWith("signup", { email: "new@test.com" });
});
```

## Async Testing Patterns

### Promises and async/await

```typescript
it("fetches user data from API", async () => {
  const api = createMockApi({ users: [{ id: "1", name: "Alice" }] });
  const service = new UserService(api);

  const user = await service.getUser("1");

  expect(user.name).toBe("Alice");
});
```

### Testing Rejected Promises

```typescript
it("throws NotFoundError for missing user", async () => {
  const api = createMockApi({ users: [] });
  const service = new UserService(api);

  await expect(service.getUser("999")).rejects.toThrow(NotFoundError);
});
```

### Testing with Timers

```typescript
import { vi, beforeEach, afterEach } from "vitest";

beforeEach(() => vi.useFakeTimers());
afterEach(() => vi.useRealTimers());

it("retries after 1 second on failure", async () => {
  const api = { fetch: vi.fn().mockRejectedValueOnce(new Error()).mockResolvedValue("ok") };
  const service = new RetryingService(api);

  const promise = service.fetchWithRetry();
  
  await vi.advanceTimersByTimeAsync(1000);
  
  expect(await promise).toBe("ok");
  expect(api.fetch).toHaveBeenCalledTimes(2);
});
```

## Error Handling Tests

```typescript
describe("validateEmail", () => {
  it("throws ValidationError for missing @ symbol", () => {
    expect(() => validateEmail("invalid")).toThrow(ValidationError);
    expect(() => validateEmail("invalid")).toThrow("must contain @");
  });

  it("throws ValidationError for empty string", () => {
    expect(() => validateEmail("")).toThrow(ValidationError);
  });

  it("returns normalized email for valid input", () => {
    expect(validateEmail("USER@Example.COM")).toBe("user@example.com");
  });
});
```

## Test Organization

### File Structure

```
src/
├── services/
│   └── UserService.ts
├── utils/
│   └── validation.ts
tests/
├── unit/
│   ├── services/
│   │   └── UserService.test.ts
│   └── utils/
│       └── validation.test.ts
└── integration/
    └── api/
        └── users.integration.test.ts
```

### Grouping Related Tests

```typescript
// UserService.test.ts
describe("UserService", () => {
  // Setup shared across all tests in this file
  const createService = (deps: Partial<Dependencies> = {}) => {
    return new UserService({
      userRepo: new FakeUserRepo(),
      emailService: { send: vi.fn() },
      ...deps,
    });
  };

  describe("registration", () => {
    it("creates user with hashed password", async () => { /* ... */ });
    it("sends verification email", async () => { /* ... */ });
    it("rejects duplicate emails", async () => { /* ... */ });
  });

  describe("authentication", () => {
    it("returns token for valid credentials", async () => { /* ... */ });
    it("throws for invalid password", async () => { /* ... */ });
    it("locks account after 5 failed attempts", async () => { /* ... */ });
  });
});
```
