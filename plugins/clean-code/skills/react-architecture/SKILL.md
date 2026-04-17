---
name: react-architecture
description: Feature-based React app architecture with separation of hooks (React state/lifecycle), services (business logic/API), and components. Use when building scalable React applications that need clear separation of concerns, maintainable code organization, and testable business logic.
---

# React Feature-Based Architecture

Organize React apps by features with clear separation between React concerns and business logic.

## Directory Structure

```
src/
├── features/              # Feature modules (self-contained)
│   ├── auth/
│   │   ├── components/    # Feature UI components
│   │   ├── hooks/         # React state/lifecycle
│   │   ├── services/      # Business logic/API
│   │   ├── types/         # TypeScript types
│   │   └── index.ts       # Public API
│   └── shared/            # Shared across features
├── core/                  # Framework-agnostic
│   ├── api/              # API client setup
│   ├── services/         # Global services
│   └── types/            # Global types
├── ui/                    # Pure presentational
│   ├── components/       # Reusable UI components
│   └── hooks/            # UI-only hooks
└── app/                  # App bootstrap
```

## Core Principles

### Separation of Concerns

**Hooks**: React-specific state and lifecycle
```typescript
// features/email/hooks/useEmails.ts
function useEmails(filters?: EmailFilter) {
  const [emails, setEmails] = useState<Email[]>([]);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    EmailService.fetchEmails(filters)
      .then(setEmails)
      .finally(() => setLoading(false));
  }, [filters]);
  
  return { emails, loading };
}
```

**Services**: Business logic and API calls
```typescript
// features/email/services/emailService.ts
export class EmailService {
  static async fetchEmails(filters?: EmailFilter): Promise<Email[]> {
    const response = await apiClient.get('/emails', { params: filters });
    return response.data.map(transformEmail);
  }
  
  static async sendEmail(payload: EmailPayload): Promise<Email> {
    const response = await apiClient.post('/emails', payload);
    return transformEmail(response.data);
  }
}
```

### Feature Module Pattern

Each feature exports its public API:
```typescript
// features/email/index.ts
export { EmailList, EmailComposer } from './components';
export { useEmails, useEmailDraft } from './hooks';
export { EmailService } from './services';
export type { Email, EmailFilter } from './types';
```

### Decision Rules

**Put in Hooks:**
- Component state management
- React lifecycle logic
- Context subscriptions
- UI-specific computations
- Orchestrating service calls with React

**Put in Services:**
- API communication
- Business rules and validation
- Data transformations
- Complex calculations
- Anything testable without React

### API Client Setup

Centralized API configuration:
```typescript
// core/api/client.ts
class ApiClient {
  private client: AxiosInstance;
  
  constructor() {
    this.client = axios.create({
      baseURL: import.meta.env.VITE_API_URL,
      timeout: 10000,
    });
    setupInterceptors(this.client);
  }
  
  async get<T>(url: string, config?: any): Promise<T> {
    const { data } = await this.client.get<T>(url, config);
    return data;
  }
}

export const apiClient = new ApiClient();
```

### Path Aliases

Configure in tsconfig.json:
```json
{
  "compilerOptions": {
    "paths": {
      "@/*": ["./src/*"],
      "@features/*": ["./src/features/*"],
      "@core/*": ["./src/core/*"],
      "@ui/*": ["./src/ui/*"]
    }
  }
}
```

## Quick Checklist

- ✅ Features are self-contained modules
- ✅ Public APIs control what's exposed
- ✅ Hooks handle React, services handle business
- ✅ Core layer has no React dependencies
- ✅ UI components are purely presentational
- ✅ Types are co-located with features
- ✅ Services are testable without React

## Common Patterns

### Async Operations Hook
```typescript
function useAsyncOperation<T>() {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<Error | null>(null);
  
  const execute = async (asyncFn: () => Promise<T>) => {
    setLoading(true);
    try {
      const result = await asyncFn();
      setData(result);
      return result;
    } catch (err) {
      setError(err as Error);
      throw err;
    } finally {
      setLoading(false);
    }
  };
  
  return { data, loading, error, execute };
}
```

### Repository Pattern (Optional)
```typescript
interface EmailRepository {
  findAll(filters?: EmailFilter): Promise<Email[]>;
  findById(id: string): Promise<Email>;
  create(email: EmailPayload): Promise<Email>;
}

class SupabaseEmailRepository implements EmailRepository {
  // Implementation specific to Supabase
}
```

## Testing Structure

Mirror source structure:
```
src/features/email/hooks/useEmails.ts
tests/features/email/hooks/useEmails.test.ts
```

## When to Apply

Use this architecture when:
- Building apps with 5+ features
- Multiple developers on the team
- Business logic complexity is moderate to high
- Testing is a priority
- Future maintainability matters

For simple apps (1-3 features), a flatter structure may suffice.
