# Service Integration Patterns

*This file documents service integration patterns and implementations within the `/src/libs` directory - the centralized service configuration hub for NextJS-BP-SaaS.*

## Service Integration Architecture

The `/src/libs` directory implements a **service integration hub** pattern with 7 specialized services following consistent factory patterns, type-safe configuration, and production-ready initialization strategies. Each service encapsulates a single external integration while maintaining consistent interfaces and error handling patterns.

### Core Integration Services

- **`Env.ts`**: Type-safe environment validation foundation using @t3-oss/env-nextjs
- **`DB.ts`**: Database connection management with Drizzle ORM and hot-reload protection  
- **`Arcjet.ts`**: Security middleware with bot protection and rate limiting
- **`Logger.ts`**: Structured logging with multi-sink support (LogTape)
- **`I18n.ts`**: Internationalization configuration (next-intl)
- **`I18nRouting.ts`**: Locale-based routing configuration
- **`I18nNavigation.ts`**: Internationalized navigation helpers

## Implementation Patterns

### 1. Environment-First Configuration Pattern

**Foundation Service**: `Env.ts` provides type-safe environment validation for all other services.

```typescript
// Hierarchical environment configuration
export const Env = createEnv({
  server: {
    ARCJET_KEY: z.string().startsWith('ajkey_').optional(),
    CLERK_SECRET_KEY: z.string().min(1),
    DATABASE_URL: z.string().min(1),
  },
  client: {
    NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY: z.string().min(1),
    NEXT_PUBLIC_POSTHOG_KEY: z.string().optional(),
  },
  shared: {
    NODE_ENV: z.enum(['test', 'development', 'production']).optional(),
  },
});
```

**Key Features**:
- **Server/Client/Shared Separation**: Clear boundaries for environment variable access
- **Format Validation**: Custom Zod validators (e.g., ARCJET_KEY must start with 'ajkey_')
- **Runtime Safety**: Validates at application startup with TypeScript inference
- **Single Source of Truth**: All other services consume validated environment variables

### 2. Singleton with Hot-Reload Protection Pattern

**Database Connection Management**: `DB.ts` implements singleton pattern with separated connection factory for modular architecture.

```typescript
// DB.ts - Global instance management
import { createDbConnection } from '@/utils/DBConnection';

const globalForDb = globalThis as unknown as {
  drizzle: NodePgDatabase<typeof schema>;
};

const db = globalForDb.drizzle || createDbConnection();

// Development-only global storage prevents connection storms
if (Env.NODE_ENV !== 'production') {
  globalForDb.drizzle = db;
}

// DBConnection.ts - Connection factory utility
export const createDbConnection = () => {
  const pool = new Pool({
    connectionString: Env.DATABASE_URL,
    ssl: !Env.DATABASE_URL.includes('localhost') && !Env.DATABASE_URL.includes('127.0.0.1'),
    max: 1,
  });

  return drizzle({ client: pool, schema });
};
```

**Implementation Benefits**:
- **Hot-Reload Protection**: Prevents multiple database connections during development
- **SSL Auto-Detection**: Automatically configures SSL based on connection string (localhost/127.0.0.1 detection)
- **Environment-Aware**: Different connection caching behavior for development vs production  
- **Schema Integration**: Type-safe database operations with `/src/models/Schema.ts`
- **Modular Architecture**: Separated connection factory allows reuse in migrations and testing
- **Connection Pooling**: Uses pg.Pool with single connection limit for controlled resource usage

### 3. Service Factory with Extension Pattern

**Security Service Configuration**: `Arcjet.ts` provides extensible base security configuration.

```typescript
export default arcjet({
  key: process.env.ARCJET_KEY ?? '', // Direct process.env for bundle optimization
  characteristics: ['ip.src'],
  rules: [
    shield({
      mode: 'LIVE', // Production-ready default
    }),
  ],
});
```

**Extension in Middleware**:
```typescript
// Route-specific security rules extend base configuration
const aj = arcjet({
  rules: [
    botDetection({
      mode: 'LIVE',
      allow: ['CATEGORY:SEARCH_ENGINE', 'CATEGORY:PREVIEW', 'CATEGORY:MONITOR'],
    }),
  ],
});
```

**Pattern Advantages**:
- **Bundle Size Optimization**: Direct `process.env` access in middleware reduces bundle size
- **Composable Rules**: Base configuration extended with route-specific security rules
- **Production-Ready Defaults**: Services default to secure configurations (LIVE mode)

### 4. Conditional Service Activation Pattern

**Multi-Sink Logging Configuration**: `Logger.ts` enables services based on environment configuration.

```typescript
const betterStackSink = fromBetterStack({
  sourceToken: Env.NEXT_PUBLIC_BETTER_STACK_SOURCE_TOKEN!,
  host: Env.NEXT_PUBLIC_BETTER_STACK_INGESTING_HOST!,
});

await configure({
  sinks: {
    console: getConsoleSink(),
    ...(Env.NEXT_PUBLIC_BETTER_STACK_SOURCE_TOKEN && 
        Env.NEXT_PUBLIC_BETTER_STACK_INGESTING_HOST && {
      betterStack: fromAsyncSink(betterStackSink),
    }),
  },
  loggers: [{
    category: ['app'],
    sinks: Env.NEXT_PUBLIC_BETTER_STACK_SOURCE_TOKEN ? 
           ['console', 'betterStack'] : 
           ['console'],
    lowestLevel: 'debug',
  }],
});
```

**Graceful Degradation Benefits**:
- **Feature Flag Pattern**: Services enable additional features when environment variables present
- **Fallback Strategy**: Console logging always available, external services optional
- **Development Experience**: Full logging in development, enhanced logging in production

## Key Files and Structure

### Environment and Configuration Files

- **`Env.ts`**: Foundation service providing type-safe environment variable validation
  - **Dependencies**: None (foundation service)
  - **Exports**: Validated environment variables with TypeScript inference
  - **Pattern**: @t3-oss/env-nextjs with Zod schema validation
  - **Usage**: All other services depend on this for configuration

- **`DB.ts`**: Database connection and schema management
  - **Dependencies**: `Env.ts` (via DBConnection), `/src/utils/DBConnection.ts`, `/src/models/Schema.ts`
  - **Exports**: Drizzle database instance with connection pooling
  - **Pattern**: Singleton with global storage using separated connection factory
  - **Features**: Hot-reload protection, environment-aware caching, modular connection management

### External Service Integrations

- **`Arcjet.ts`**: Security and bot protection service
  - **Dependencies**: Direct `process.env` access (bundle size optimization)
  - **Exports**: Base security configuration for extension
  - **Pattern**: Middleware-level bot detection with IP characteristics
  - **Features**: Shield protection, extensible rule system, LIVE mode default

- **`Logger.ts`**: Structured logging with multiple output sinks
  - **Dependencies**: `Env.ts` (Better Stack configuration)
  - **Exports**: Configured logger instance with category-based routing
  - **Pattern**: Multi-sink logging with conditional external service activation
  - **Features**: Console + Better Stack logging, async sink support

### Internationalization Services

- **`I18n.ts`**: Main internationalization configuration
  - **Dependencies**: `I18nRouting.ts`, dynamic locale imports
  - **Exports**: next-intl configuration with server-side rendering
  - **Pattern**: Dynamic message loading with locale validation
  - **Features**: Crowdin integration, fallback locale handling

- **`I18nRouting.ts`**: Locale routing configuration
  - **Dependencies**: `/src/utils/AppConfig.ts` (locale definitions)
  - **Exports**: Routing configuration for next-intl
  - **Pattern**: As-needed locale prefix with validation
  - **Features**: English/French support, locale-aware pathnames

- **`I18nNavigation.ts`**: Navigation helpers for internationalization
  - **Dependencies**: `I18nRouting.ts`
  - **Exports**: Type-safe navigation utilities
  - **Pattern**: Wrapped Next.js navigation with locale handling
  - **Features**: Automatic locale detection, type-safe routing

## Integration Points

### Middleware Orchestration Pattern

**Service Coordination**: `/src/middleware.ts` orchestrates multiple service layers in sequence.

```typescript
// Processing pipeline: Security → Authentication → Internationalization
1. Arcjet Security: Bot detection, rate limiting, shield protection
2. Clerk Authentication: Route-based conditional authentication  
3. I18n Routing: Locale resolution and routing
```

**Integration Benefits**:
- **Layered Security**: Multiple security layers with different focuses
- **Performance Optimization**: Early request rejection reduces downstream processing
- **Locale-Aware Authentication**: Authentication URLs respect user's locale preferences

### Database and Logging Integration

**Consistent Error Handling**: Database operations integrate structured logging throughout the application.

```typescript
// Pattern used in API routes and components
import { db } from '@/libs/DB';
import { logger } from '@/libs/Logger';

try {
  const result = await db.insert(schema).values(data);
  logger.info('Database operation completed', { operation: 'insert', table: 'example' });
  return result;
} catch (error) {
  logger.error('Database operation failed', { error, operation: 'insert' });
  throw error;
}
```

### Component Integration Patterns

**Service Consumption**: Application components consistently consume configured services.

- **Database**: API routes use `db` for type-safe database operations
- **Logging**: Components use `logger` for structured application logging  
- **Environment**: Components access validated environment variables via `Env`
- **I18n**: Components use navigation helpers for locale-aware routing

## Development Patterns

### Service Health and Monitoring

**Integrated Observability**: Services integrate with monitoring systems for production readiness.

- **Sentry Integration**: Automatic error capture via `/src/instrumentation.ts`
- **Request Correlation**: Structured logging with correlation IDs for debugging
- **Performance Monitoring**: Database query timing and external service response tracking
- **Security Events**: Arcjet security events logged for analysis

### Testing and Development Workflow

**Service Testing Patterns**:

- **Environment Validation**: Test environment variable validation with invalid configurations
- **Database Integration**: Use PGLite for development testing without Docker dependencies
- **Service Mocking**: Mock external services (Better Stack, Arcjet) for unit testing
- **Internationalization**: Test locale routing and translation loading patterns

**Development Experience Features**:

- **Hot Reload Compatibility**: Database connections cached to prevent reconnection storms
- **Development Debugging**: Spotlight integration for Sentry development debugging
- **Environment Detection**: Services provide enhanced development logging and debugging
- **Type Safety**: Full TypeScript integration with comprehensive type checking

### Configuration Management Best Practices

**Service Configuration Guidelines**:

1. **Environment-First**: Always validate configuration through `Env.ts`
2. **Graceful Degradation**: Services should work with minimal required configuration
3. **Production Defaults**: Services default to production-ready, secure configurations
4. **Development Enhancement**: Additional features enabled in development environments
5. **Type Safety**: All configuration values validated at runtime with TypeScript inference

---

*This service integration layer demonstrates enterprise-grade patterns for external service management with comprehensive type safety, monitoring integration, and development experience optimization. For architectural context, see `/src/CONTEXT.md`. For technology stack details, see `/docs/ai-context/project-structure.md`.*