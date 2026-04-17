---
name: trpc
description: Architecture patterns and code generation for tRPC applications in monorepo setups. Use when building tRPC routers, services, repositories in a monorepo with proper separation of concerns, creating new features with tRPC, refactoring to layered architecture, or setting up shared packages between frontend and backend.
---

### Error Handling Pattern

```typescript
// Service throws custom errors:
throw new NotFoundError('User not found');

// Router doesn't catch - let tRPC middleware handle transformation
```
