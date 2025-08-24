# Input Validation Documentation

## Input Validation Architecture

**Core Framework**: Zod 3.x with TypeScript integration providing compile-time and runtime validation across the Next.js application stack.

**Architectural Strategy**: Shared validation schemas between client and server components ensuring type consistency, business rule enforcement, and centralized validation logic.

```typescript
// Centralized validation schema with business rules
export const CounterValidation = z.object({
  increment: z.coerce.number().min(1).max(3),
});
```

**Key Design Principles**:
- **Single Source of Truth**: One schema definition per data structure
- **Cross-Stack Consistency**: Same validation rules for client forms and API endpoints
- **Type Safety**: Automatic TypeScript type inference from Zod schemas
- **Business Rule Enforcement**: Domain constraints embedded in validation layer

## Implementation Patterns

### Server-Side API Validation Pattern
```typescript
// API route validation with structured error handling
export const PUT = async (request: Request) => {
  const json = await request.json();
  const parse = CounterValidation.safeParse(json);

  if (!parse.success) {
    return NextResponse.json(z.treeifyError(parse.error), { status: 422 });
  }

  // Type-safe access to validated data
  const validatedData = parse.data; // { increment: number }
};
```

**Server-Side Patterns**:
- **`safeParse()` Method**: Non-throwing validation with success/error discrimination
- **Structured Error Response**: `z.treeifyError()` provides consistent API error format
- **HTTP Status Integration**: 422 status code for validation failures
- **Type-Safe Data Access**: `parse.data` provides fully typed validated data

### Client-Side Form Integration Pattern
```typescript
// React Hook Form + Zod integration via zodResolver
const form = useForm({
  resolver: zodResolver(CounterValidation),
  defaultValues: {
    increment: 0,
  },
});
```

**Client-Side Patterns**:
- **`zodResolver` Integration**: Seamless React Hook Form validation
- **Real-Time Validation**: Form validation occurs on input change
- **Error State Management**: Form errors automatically derived from schema violations
- **Default Value Typing**: Type-safe default values based on schema structure

### Environment Configuration Validation
```typescript
// Environment variable validation with type coercion
const envSchema = z.object({
  DATABASE_URL: z.string().min(1),
  NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY: z.string().min(1),
  ARCJET_KEY: z.string().startsWith('ajkey_').optional(),
  NEXT_PUBLIC_POSTHOG_KEY: z.string().optional(),
});
```

**Environment Patterns**:
- **Configuration Validation**: Runtime validation of environment variables
- **String Constraints**: `startsWith()`, `min()` for format and presence validation
- **Optional Fields**: `.optional()` for non-required configuration
- **Type Inference**: Automatic TypeScript types for environment configuration

## Key Files and Structure

### Current Schema Organization
```
src/validations/
└── CounterValidation.ts     # Counter API and form validation schema
```

**Schema Structure**:
- **Single Export Pattern**: Each file exports one primary validation schema
- **Business Rule Encapsulation**: Domain constraints (min: 1, max: 3) defined at schema level
- **Type Coercion**: `z.coerce.number()` handles string-to-number conversion from forms
- **Descriptive Naming**: Schema names match their primary use case (Counter operations)

### Schema Definition Anatomy
```typescript
import { z } from 'zod';

export const CounterValidation = z.object({
  increment: z.coerce.number()  // Type coercion for form inputs
    .min(1)                     // Business rule: minimum increment
    .max(3),                    // Business rule: maximum increment
});

// Automatic TypeScript type inference:
// type CounterValidationType = {
//   increment: number;
// }
```

## Integration Points

### API Route Integration (`/src/app/[locale]/api/counter/route.ts`)
```typescript
// Validation pipeline in API routes
const parse = CounterValidation.safeParse(json);
if (!parse.success) {
  return NextResponse.json(z.treeifyError(parse.error), { status: 422 });
}

// Direct usage of validated data in database operations
const count = await db.insert(counterSchema).values({
  id, 
  count: parse.data.increment  // Type-safe, validated data
});
```

**API Integration Patterns**:
- **Request Validation Gate**: All incoming data validated before processing
- **Database Operation Safety**: Only validated data reaches database operations
- **Consistent Error Format**: Standardized error responses across all API endpoints
- **Type Safety Chain**: Validation → Database → Response maintains type safety

### Component Form Integration (`/src/components/CounterForm.tsx`)
```typescript
// Form validation integration with React Hook Form
const form = useForm({
  resolver: zodResolver(CounterValidation),
  defaultValues: { increment: 0 },
});

// Automatic error handling
{form.formState.errors.increment && (
  <div className="text-red-500">
    {t('error_increment_range')}
  </div>
)}
```

**Component Integration Patterns**:
- **Automatic Validation**: Form inputs validated against schema on change/submit
- **Error State Binding**: Form errors automatically mapped from schema violations
- **Internationalization Ready**: Error messages integrated with i18n system
- **Accessibility Support**: Error states properly connected to form controls

### Cross-Stack Type Consistency
```typescript
// Same schema used across multiple layers:
// 1. Client-side form (via zodResolver)
// 2. API validation (via safeParse)  
// 3. Database operations (via parse.data)
// 4. TypeScript inference (automatic types)
```

**Type Consistency Benefits**:
- **Compile-Time Validation**: Schema changes surface type errors immediately
- **Refactoring Safety**: Schema modifications automatically update all usage points
- **Developer Experience**: IntelliSense and autocomplete across entire validation chain
- **Runtime Safety**: Same validation logic prevents client/server data inconsistencies

## Development Patterns

### Schema Evolution Workflow
1. **Modify Schema**: Update validation rules in schema file
2. **TypeScript Check**: Compilation errors identify affected code
3. **Update Usage**: Modify forms, API routes, and components as needed
4. **Test Integration**: Verify client-side and server-side validation behavior
5. **Error Message Updates**: Update internationalization keys for new validation rules

### Error Handling Strategy
```typescript
// Comprehensive error handling pattern
const result = CounterValidation.safeParse(input);

if (!result.success) {
  // Server-side: Structured API response
  return NextResponse.json(z.treeifyError(result.error), { status: 422 });
  
  // Client-side: Form error state automatically handled by zodResolver
}

// Success path: Type-safe data access
const validatedData: { increment: number } = result.data;
```

**Error Handling Patterns**:
- **Non-Throwing Validation**: `safeParse()` prevents unhandled validation exceptions
- **Discriminated Unions**: `success` property enables safe error/success handling
- **Structured Error Objects**: Consistent error format across validation failures
- **Client-Server Consistency**: Same error handling patterns in forms and APIs

### Testing and Debugging
```typescript
// Schema testing pattern
describe('CounterValidation', () => {
  it('accepts valid increment values', () => {
    const result = CounterValidation.safeParse({ increment: 2 });
    expect(result.success).toBe(true);
  });

  it('rejects out-of-range values', () => {
    const result = CounterValidation.safeParse({ increment: 5 });
    expect(result.success).toBe(false);
  });
});
```

**Development Support Patterns**:
- **Unit Testable**: Schemas can be tested independently of components/APIs
- **Clear Error Messages**: Validation failures provide specific field-level feedback
- **Debug-Friendly**: `safeParse` results include detailed error information
- **Schema Introspection**: Zod schemas provide runtime type information for tooling

### Business Rule Management
```typescript
// Business rules defined at validation layer
export const CounterValidation = z.object({
  increment: z.coerce.number()
    .min(1, "Increment must be at least 1")     // Business constraint
    .max(3, "Increment cannot exceed 3"),       // Business constraint
});
```

**Business Rule Integration**:
- **Centralized Rules**: Domain constraints defined once, enforced everywhere
- **Custom Error Messages**: Business-friendly error messages for rule violations
- **Rule Documentation**: Schema serves as living documentation of business constraints
- **Consistency Enforcement**: Same rules applied to forms, APIs, and data processing

## System Integration Context

### Cross-Component Dependencies
- **Environment Validation** (`/src/libs/Env.ts`): Zod schemas validate configuration
- **API Routes** (`/src/app/[locale]/api/*/route.ts`): Server-side request validation
- **Form Components** (`/src/components/*Form.tsx`): Client-side input validation
- **Database Operations**: Validated data used in Drizzle ORM operations
- **Internationalization**: Error messages integrated with next-intl system

### Future Expansion Patterns
- **New Schema Addition**: Follow `*Validation.ts` naming convention
- **Complex Validations**: Use Zod's advanced features (refinements, transforms, conditionals)
- **Schema Composition**: Combine smaller schemas using `z.intersection()` or `z.union()`
- **Custom Validators**: Extend with `.refine()` for complex business logic

This validation system provides type-safe, consistent data validation across the entire Next.js application stack while maintaining clear separation of concerns and enabling rapid feature development.