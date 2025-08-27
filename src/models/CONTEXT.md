# Database Schema - Implementation Context

## Overview
**Architectural Approach**: Centralized schema definition using Drizzle ORM 0.44.4 with PostgreSQL-native types, designed for type-safe database operations across the Next.js application stack.

**Key Philosophy**: Single source of truth for database schema with automatic TypeScript inference, evolution-friendly design patterns, and seamless integration with server components, API routes, and validation layers.

## Core Architecture Patterns

### Schema Definition (`Schema.ts`)
```typescript
// Centralized schema with PostgreSQL-native types
export const counterSchema = pgTable('counter', {
  id: serial('id').primaryKey(),
  count: integer('count').default(0),
  updatedAt: timestamp('updated_at', { mode: 'date' })
    .defaultNow()
    .$onUpdate(() => new Date()) // Automatic timestamp updates
    .notNull(),
  createdAt: timestamp('created_at', { mode: 'date' }).defaultNow().notNull(),
});
```

**Design Patterns**:
- **PostgreSQL-First**: Uses `pgTable`, `serial`, `integer`, `timestamp` for optimal PostgreSQL compatibility
- **Automatic Timestamps**: `$onUpdate()` provides automatic `updatedAt` field management
- **Type Safety**: Full TypeScript inference for all database operations
- **Evolution-Friendly**: Schema changes automatically generate migration files

### Database Connection Management (`/src/libs/DB.ts`)
```typescript
// Singleton with hot-reload protection
const globalForDb = globalThis as unknown as {
  drizzle: NodePgDatabase<typeof schema>;
};

const createDbConnection = () => {
  return drizzle({
    connection: {
      connectionString: Env.DATABASE_URL,
      ssl: !Env.DATABASE_URL.includes('localhost') && !Env.DATABASE_URL.includes('127.0.0.1'),
    },
    schema, // Full schema import for type inference
  });
};

// Development hot-reload protection
const db = globalForDb.drizzle || createDbConnection();
if (Env.NODE_ENV !== 'production') {
  globalForDb.drizzle = db;
}
```

**Implementation Patterns**:
- **Singleton Pattern**: Single database connection per application instance
- **Hot-Reload Protection**: Prevents multiple connections during Next.js development
- **Environment-Based SSL**: Automatic SSL configuration based on connection string
- **Schema Integration**: Full schema import provides complete type inference

## Migration Management

### Migration Generation and Application
```typescript
// Migration execution (DBMigration.ts)
await migrate(db, {
  migrationsFolder: path.join(process.cwd(), 'migrations'),
});
```

**Migration Strategy**:
- **Generated Migrations**: `npm run db:generate` creates SQL migration files from schema changes
- **Automatic Application**: Migrations run during Next.js initialization via `instrumentation.ts`
- **Development Workflow**: Schema changes → Generate → Restart server (automatic application)
- **Production Workflow**: `npm run db:migrate` for manual migration execution

### Generated Migration Example
```sql
-- 0000_init-db.sql
CREATE TABLE "counter" (
  "id" serial PRIMARY KEY NOT NULL,
  "count" integer DEFAULT 0,
  "updated_at" timestamp DEFAULT now() NOT NULL,
  "created_at" timestamp DEFAULT now() NOT NULL
);
```

## Usage Patterns Across Stack

### API Route Integration (`/src/app/[locale]/api/counter/route.ts`)
```typescript
// Complex upsert with SQL expressions
const count = await db
  .insert(counterSchema)
  .values({ id, count: parse.data.increment })
  .onConflictDoUpdate({
    target: counterSchema.id,
    set: { count: sql`${counterSchema.count} + ${parse.data.increment}` },
  })
  .returning();
```

**API Patterns**:
- **Upsert Operations**: `onConflictDoUpdate` for atomic increment operations
- **SQL Expressions**: Direct SQL for complex operations (`sql` template literal)
- **Validation Integration**: Zod schema validation before database operations
- **Header-Based Testing**: `x-e2e-random-id` header for isolated E2E test data
- **Type-Safe Returns**: Full TypeScript inference on returned data

### Server Component Usage (`/src/components/CurrentCount.tsx`)
```typescript
// Query with conditional access
const result = await db.query.counterSchema.findMany({
  where: eq(counterSchema.id, id),
});
const count = result[0]?.count ?? 0;
```

**Component Patterns**:
- **Query Builder**: `db.query.counterSchema` for type-safe queries
- **Conditional Access**: Safe access with nullish coalescing for missing records
- **Header Integration**: E2E testing support via request headers
- **Server-Side Execution**: Direct database access in React Server Components

### Validation Coordination (`/src/validations/CounterValidation.ts`)
```typescript
// Coordinated validation schema
export const CounterValidation = z.object({
  increment: z.coerce.number().min(1).max(3),
});
```

**Validation Patterns**:
- **Business Rule Enforcement**: Min/max constraints aligned with application logic
- **Type Coercion**: Automatic number conversion from form/API inputs
- **API Integration**: Validation occurs before database operations in API routes

## Development Workflow

### Schema Evolution Process
1. **Modify Schema**: Update `Schema.ts` with new fields, tables, or relationships
2. **Generate Migration**: Run `npm run db:generate` to create migration files
3. **Review Migration**: Examine generated SQL in `migrations/` directory
4. **Apply Changes**: Restart Next.js server (automatic) or run `npm run db:migrate`
5. **Update Usage**: Modify components, API routes, and validation schemas as needed

### Type Safety Workflow
```typescript
// Full type inference throughout stack
const result: {
  id: number;
  count: number | null;
  updatedAt: Date;
  createdAt: Date;
}[] = await db.query.counterSchema.findMany();
```

**Type Safety Benefits**:
- **Compile-Time Validation**: Schema changes immediately surface type errors
- **Intellisense Support**: Full autocomplete for fields and operations
- **Refactoring Safety**: Field renames automatically update throughout codebase
- **Runtime Safety**: Drizzle validates operations against actual schema

## Security and Performance Considerations

### Security Patterns
- **Parameterized Queries**: All operations use parameterized queries (SQL injection protection)
- **Input Validation**: Zod validation prevents invalid data from reaching database
- **Environment-Based Configuration**: Sensitive connection details via environment variables
- **SSL Configuration**: Automatic SSL for production database connections

### Performance Optimizations
- **Connection Pooling**: Single connection instance with hot-reload protection
- **Selective Queries**: Query builder supports precise field selection
- **SQL Expression Support**: Raw SQL for performance-critical operations
- **Timestamp Automation**: Database-level timestamp management reduces application overhead

### Testing Integration
- **E2E Isolation**: Header-based ID system enables isolated test data
- **Migration Testing**: Generated migrations provide testable schema changes
- **Type-Safe Mocking**: Schema types enable accurate test data structures
- **Development Database**: Local database setup for development and testing

## Integration Points

### Next.js Integration
- **Server Components**: Direct database access in React Server Components
- **API Routes**: Type-safe database operations in route handlers
- **Middleware**: Schema available for middleware-level data operations
- **Instrumentation**: Automatic migration application during server startup

### External Service Coordination
- **Validation Layer**: Coordinated with Zod schemas in `/src/validations/`
- **Logging Integration**: Database operations logged via centralized Logger
- **Environment Management**: Database configuration via centralized Env validation
- **Error Handling**: Type-safe error handling with NextResponse integration

This implementation provides a robust foundation for database operations with strong type safety, automated migration management, and seamless integration across the Next.js application stack.
