# NextJS-BP-SaaS Source Code Architecture

## Purpose
The `/src` directory implements a production-ready Next.js 15 application with enterprise-grade features including authentication, internationalization, security, analytics, and comprehensive observability. This codebase demonstrates modern full-stack development patterns with AI-first development integration.

## Current Status: Production Ready
This source architecture has achieved production readiness with comprehensive testing, security hardening, type safety, and monitoring. The codebase follows strict quality gates including ESLint validation, TypeScript checking, automated testing, and security scanning.

## Component-Specific Development Guidelines

### TypeScript-First Development
- **Strict Configuration**: All code must pass TypeScript strict mode checking
- **Runtime Validation**: Use Zod schemas for input validation and environment variables
- **Type-Safe Database**: Drizzle ORM provides compile-time SQL type checking
- **End-to-End Types**: Maintain type safety from database schema to React components

### Security-First Implementation
- **Defense in Depth**: Multiple security layers (Arcjet → Clerk → API validation)
- **Environment Validation**: All environment variables validated with Zod schemas
- **Input Sanitization**: Validate all inputs at API boundaries using Zod
- **Error Security**: Never expose internal system details in error responses

### Next.js 15 App Router Patterns
- **Server Components**: Default to server components, use client components only when necessary
- **Route Groups**: Use `(auth)`, `(marketing)`, `(center)` for logical organization without URL impact
- **Dynamic Routing**: Implement `[locale]` for internationalization and `[slug]` for dynamic content
- **Metadata API**: Generate SEO-optimized metadata using the metadata API

### Internationalization Standards
- **Locale-First Routing**: All routes must support `[locale]` dynamic segments
- **Server-Side Translation**: Use `getTranslations()` in server components
- **Client-Side Translation**: Use `useTranslations()` hook in client components
- **Type-Safe Translations**: Leverage next-intl's TypeScript integration

## Major Subsystem Organization

### `/src/app/` - Next.js App Router Implementation
**Route Structure with Internationalization**
```
/src/app/[locale]/
├── (auth)/          # Protected routes requiring authentication
│   ├── (center)/    # Centered layout authentication pages
│   └── dashboard/   # Main application interface
├── (marketing)/     # Public marketing pages
└── api/            # RESTful API endpoints
```

**Key Patterns:**
- Route groups provide logical organization without affecting URLs
- All pages implement metadata generation for SEO optimization
- Server-side locale handling with `setRequestLocale(locale)`
- Consistent layout hierarchy: root → locale → route group → page

### `/src/components/` - React Component System
**Component Architecture**
- **Atomic Components**: Single-purpose, reusable components
- **Composed Components**: Complex components built from atomic parts
- **Analytics Integration**: PostHog tracking components for user behavior
- **Accessibility-First**: All components follow WCAG guidelines

**Testing Strategy**: Each template component includes `.test.tsx` and `.stories.tsx` files

### `/src/libs/` - Service Integration Hub
**Infrastructure Services**
- **`Arcjet.ts`**: Security middleware with bot protection and rate limiting
- **`DB.ts`**: Database connection management with hot-reload protection  
- **`Env.ts`**: Type-safe environment variable validation using @t3-oss/env-nextjs
- **`I18n.ts`**: Internationalization configuration with next-intl
- **`Logger.ts`**: Structured logging with LogTape supporting multiple outputs

**Integration Pattern**: All external services configured here and exported as singletons

### `/src/models/` - Database Schema Layer
**Drizzle ORM Implementation**
- **Schema Definition**: PostgreSQL schemas with TypeScript types
- **Migration Management**: Automatic execution via instrumentation
- **Development Database**: PGLite for local development without Docker

### `/src/templates/` - Layout System
**Template Hierarchy**
- **BaseTemplate**: Foundation page template with navigation slots
- **Route Group Layouts**: Specialized layouts for different application areas
- **Provider Composition**: Context providers at appropriate hierarchy levels

### `/src/utils/` - Utility Functions
**Helper Organization**
- **AppConfig**: Application-wide constants and configuration
- **DBMigration**: Database migration utilities
- **Helpers**: Pure utility functions with comprehensive unit tests

### `/src/validations/` - Input Validation
**Zod Schema Patterns**
- **API Validation**: Input validation for all API endpoints
- **Form Validation**: Client-side form validation with React Hook Form integration
- **Type Generation**: Zod schemas that generate TypeScript types

## Architectural Patterns

### Security Middleware Stack
**Request Processing Pipeline**
1. **Arcjet Security**: Bot detection, rate limiting, shield protection
2. **Clerk Authentication**: Route-based conditional authentication
3. **Internationalization**: Locale resolution and routing
4. **API Validation**: Zod schema validation at API boundaries

### Database Integration Pattern
```typescript
// Global connection with hot-reload protection
const globalForDb = globalThis as unknown as { drizzle: NodePgDatabase<typeof schema> };
const db = globalForDb.drizzle || createConnection();

// Development-only global storage
if (Env.NODE_ENV !== 'production') {
  globalForDb.drizzle = db;
}
```

### Environment Configuration Pattern
```typescript
// Type-safe environment validation
export const Env = createEnv({
  server: {
    CLERK_SECRET_KEY: z.string().min(1),
    DATABASE_URL: z.string().min(1),
  },
  client: {
    NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY: z.string().min(1),
  },
});
```

### Component Composition Pattern
- **Server Components**: Default for all components
- **Client Components**: Only when browser APIs or interactivity required
- **Provider Wrapping**: Context providers at layout boundaries
- **Type-Safe Props**: All components use TypeScript interfaces

## Integration Points

### Authentication Flow (Clerk)
- **Middleware Protection**: Route-based authentication guards
- **Layout Integration**: ClerkProvider at locale layout level
- **Localization**: Multi-language authentication UI
- **API Integration**: Server-side `currentUser()` for protected routes

### Database Layer (Drizzle + PostgreSQL)
- **Schema-First**: Database schema defines TypeScript types
- **Migration Strategy**: Automated migrations with instrumentation
- **Development Setup**: PGLite enables Docker-free development
- **Production Ready**: SSL connections with connection pooling

### Internationalization (next-intl)
- **Route-Based Locales**: `[locale]` dynamic routing with fallbacks
- **Server-Side Rendering**: Optimized translation loading
- **Translation Management**: Crowdin integration for translation updates
- **Type Safety**: Generated types for translation keys

### Analytics & Monitoring
- **PostHog**: Privacy-first user analytics with custom event tracking
- **Sentry**: Error tracking with source maps and performance monitoring
- **LogTape**: Structured logging with multiple output destinations
- **Checkly**: API monitoring with synthetic E2E testing

### Security Integration (Arcjet)
- **Bot Protection**: ML-powered bot detection with configurable rules
- **Rate Limiting**: IP-based protection with allowlists for legitimate services
- **Shield Protection**: Common attack protection (SQL injection, XSS, etc.)
- **Development Mode**: Conditional security rules based on environment

## Development Patterns

### Testing Strategy
- **Unit Tests**: Vitest with Node.js environment for utilities and logic
- **Component Tests**: Vitest browser environment with Playwright provider
- **E2E Tests**: Playwright with visual regression testing
- **Integration Tests**: API route testing with database interactions

### Code Quality Gates
- **TypeScript**: Strict type checking with comprehensive configuration
- **ESLint**: @antfu/eslint-config with React, Next.js, and accessibility rules
- **Git Hooks**: Lefthook pre-commit hooks for automatic code quality checks
- **Dependency Analysis**: Knip for dead code detection and dependency validation

### AI-First Development
- **Claude Integration**: Custom commands and context injection for development tasks
- **Documentation Generation**: Automated documentation updates and maintenance
- **Code Review**: AI-assisted code review workflows
- **Context Management**: Automatic project context loading for AI agents

---

*This source code architecture demonstrates enterprise-grade Next.js development with comprehensive security, internationalization, type safety, and modern development practices. All components follow consistent patterns for maintainability, scalability, and developer experience.*