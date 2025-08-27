# NextJS-BP-SaaS System Integration

This document contains cross-component integration patterns and system-wide architectural decisions for the NextJS-BP-SaaS boilerplate project.

## Security Architecture Pipeline

### Defense-in-Depth Security Stack

The system implements a three-layer security architecture that all API agents must understand:

```typescript
// Security Middleware Pipeline in /src/middleware.ts
export async function middleware(request: NextRequest) {
  // Layer 1: Arcjet Security (Bot protection, rate limiting)
  const decision = await aj.protect(request);
  if (decision.isDenied()) {
    return NextResponse.json({ error: 'Forbidden' }, { status: 403 });
  }

  // Layer 2: Clerk Authentication (Route-specific)
  const authResult = await clerkMiddleware(request);

  // Layer 3: Internationalization (Locale routing)
  return createI18nMiddleware(request);
}
```

### Security Integration Points

- **Environment Validation**: All secrets validated with Zod schemas via `@t3-oss/env-nextjs`
- **API Route Protection**: Server-side authentication using `currentUser()` never trusting client tokens
- **Input Sanitization**: All API inputs validated with Zod schemas before database operations
- **Error Boundaries**: Never expose internal errors to clients using `z.treeifyError()` pattern

## Service Integration Architecture

### Core Service Stack

**Authentication & Authorization**
- **Clerk**: Complete authentication with locale-aware URLs and internationalization support
- **Integration Pattern**: Server components use `currentUser()`, client components use `<ClerkProvider>`

**Database & ORM**
- **Drizzle ORM + PostgreSQL**: Type-safe database operations with automatic TypeScript inference
- **Hot-Reload Protection**: Database connections managed through singleton pattern in `/src/libs/DB.ts`
- **Migration Strategy**: `npm run db:generate` → `npm run db:push` workflow

**Analytics & Monitoring**
- **PostHog**: Privacy-first user analytics with server-side tracking
- **Sentry**: Error tracking and performance monitoring with source maps
- **Integration Pattern**: Context providers at layout boundaries with environment-based activation

**Logging & Observability**
- **LogTape**: Structured JSON logging with multi-sink support (console + external services)
- **Request Correlation**: User session tracking with Clerk user IDs and request context
- **Pattern**: Machine-readable logs for automated analysis and debugging

### Cross-Component Communication Patterns

**Server/Client Boundaries**
```typescript
// Server Component Pattern (Data Fetching)
const ServerComponent = async () => {
  const data = await db.query.schema.findMany();
  return <ClientComponent initialData={data} />;
};

// Client Component Pattern (Interactivity)
const ClientComponent = ({ initialData }: Props) => {
  const [data, setData] = useState(initialData);
  // Handle user interactions...
};
```

**API Route Integration**
```typescript
// Standardized API Route Pattern
export const PUT = async (request: Request): Promise<NextResponse> => {
  const json = await request.json();
  const parse = ValidationSchema.safeParse(json);

  if (!parse.success) {
    return NextResponse.json(z.treeifyError(parse.error), { status: 422 });
  }

  const result = await db.insert(schema).values(parse.data);
  return NextResponse.json(result);
};
```

## State Management Integration

**Form State Management**
- **React Hook Form + Zod**: Client-side validation with server-side confirmation
- **Pattern**: `useForm({ resolver: zodResolver(Schema) })` for consistent validation

**Global State**
- **Authentication**: Clerk providers manage auth state across components
- **Configuration**: Centralized `AppConfig` with environment-based service activation
- **Internationalization**: `next-intl` providers for locale-aware state management

## Testing Integration Strategies

**Multi-Environment Testing**
- **Unit Tests**: Vitest with component and utility testing
- **Integration Tests**: API route testing with database isolation
- **E2E Tests**: Playwright with `x-e2e-random-id` headers for test isolation
- **Visual Regression**: Storybook integration with Chromatic (future)

**Testing Pattern for Cross-Component Features**
```typescript
// Integration Test Pattern
test('Counter API integration', async () => {
  const response = await fetch('/api/counter', {
    method: 'PUT',
    headers: { 'x-e2e-random-id': 'test-123' },
    body: JSON.stringify({ increment: 1 })
  });

  expect(response.status).toBe(200);
});
```

## Performance Optimization Patterns

**Next.js Optimization**
- **App Router**: Route groups for logical organization without URL impact
- **Server Components**: Default server-side rendering with selective client hydration
- **Bundle Optimization**: Dynamic imports for code splitting and performance

**Database Optimization**
- **Connection Pooling**: Singleton database connection pattern
- **Query Optimization**: Drizzle ORM with automatic query optimization
- **Migration Management**: Evolution-friendly schema design patterns

## Error Handling Across Service Boundaries

**Structured Error Response Pattern**
```typescript
// API Error Handling
if (!validation.success) {
  return NextResponse.json(z.treeifyError(validation.error), { status: 422 });
}

// Component Error Boundaries
if (error) {
  logger.error('Component error', { context, userId });
  return <ErrorDisplay message={t('generic_error')} />;
}
```

**Cross-Service Error Propagation**
- **API Layer**: Structured error responses with status codes
- **Component Layer**: User-friendly error messages with internationalization
- **Logging Layer**: Structured error context for debugging without exposing internals

---

*This system integration documentation reflects the production-ready NextJS-BP-SaaS architecture with enterprise-grade security, monitoring, and development patterns.*
