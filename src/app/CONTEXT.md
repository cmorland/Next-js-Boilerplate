# Next.js App Router Implementation Patterns

*This file documents Next.js 15 App Router implementation patterns and routing architecture within the `/src/app` directory.*

## App Router Architecture

The `/src/app` directory implements a **Next.js 15 App Router** architecture with sophisticated route group organization, internationalization-first design, and layered authentication patterns. The implementation demonstrates enterprise-grade routing patterns with type-safe internationalization, provider composition, and modern React 19 server component architecture.

### Core Routing Structure

- **`[locale]/`**: Internationalization wrapper supporting English/French with dynamic locale routing
- **`(auth)/`**: Authentication route group with protected routes and authentication boundaries
- **`(marketing)/`**: Public marketing pages with SEO-optimized layouts  
- **`(center)/`**: Centered layout sub-group for authentication forms
- **`api/`**: RESTful API routes with validation and database integration
- **Root Files**: Global error handling, dynamic sitemap, and robots.txt generation

## Implementation Patterns

### 1. Route Group Organization Strategy

**Route Group Architecture**: Uses Next.js route groups for logical organization without URL impact.

```typescript
// Route group structure
/src/app/[locale]/
├── (auth)/                    # Authentication boundary
│   ├── (center)/              # Centered auth forms  
│   │   ├── sign-in/[[...sign-in]]/
│   │   └── sign-up/[[...sign-up]]/
│   └── dashboard/             # Protected application area
├── (marketing)/               # Public pages
└── api/                       # API endpoints
```

**Architectural Benefits**:
- **URL Cleanliness**: Route groups don't affect URLs, maintaining clean public paths
- **Layout Inheritance**: Each group has specialized layouts and provider configurations
- **Security Boundaries**: Clear separation between public, auth, and protected areas
- **Navigation Context**: Different navigation patterns per route group

### 2. Layout Composition and Provider Patterns

**Hierarchical Provider Architecture**: 4-tier layout composition with strategic provider placement.

```typescript
// Provider composition hierarchy (root to leaf)
1. Root Layout ([locale]/layout.tsx):
   - NextIntlClientProvider (internationalization)
   - PostHogProvider (analytics)
   - Global CSS and favicon configuration

2. Route Group Layouts:
   - (auth)/layout.tsx: ClerkProvider with locale-aware auth URLs
   - (marketing)/layout.tsx: BaseTemplate with public navigation
   - dashboard/layout.tsx: BaseTemplate with authenticated navigation

3. Specialized Sub-layouts:
   - (center)/layout.tsx: Simple centering wrapper for auth forms
```

**Provider Configuration Pattern**:
```typescript
// Locale-aware Clerk configuration in auth layout
const clerkLocale = ClerkLocalizations.supportedLocales[locale] ?? ClerkLocalizations.defaultLocale;
let signInUrl = locale !== routing.defaultLocale ? `/${locale}/sign-in` : '/sign-in';

<ClerkProvider
  localization={clerkLocale}
  signInUrl={signInUrl}
  signUpUrl={signUpUrl}
  signInFallbackRedirectUrl={dashboardUrl}
>
```

### 3. Internationalization Routing Mechanics

**Dynamic Locale Routing**: `[locale]` parameter provides internationalization foundation.

```typescript
// Consistent locale handling pattern across all routes
export async function generateStaticParams() {
  return routing.locales.map(locale => ({ locale }));
}

export default async function LocaleLayout(props: { children: ReactNode; params: Promise<{ locale: string }>; }) {
  const { locale } = await props.params;
  
  // Validate locale and handle 404 for unsupported locales
  if (!hasLocale(locale)) {
    notFound();
  }
  
  setRequestLocale(locale);
  // Layout implementation
}
```

**Server-Side Translation Pattern**:
```typescript
// Page-level translation loading
export async function generateMetadata(props: PageProps): Promise<Metadata> {
  const { locale } = await props.params;
  const t = await getTranslations({ locale, namespace: 'PageName' });
  
  return {
    title: t('meta_title'),
    description: t('meta_description'),
  };
}
```

**Key Features**:
- **Static Generation**: Pre-generated pages for all supported locales
- **Locale Validation**: Server-side validation with 404 fallback for invalid locales
- **SEO Optimization**: Locale-specific metadata generation for each route
- **Type Safety**: Promise-based params with TypeScript validation

### 4. Authentication Flow Implementation

**Multi-Layer Authentication Architecture**: Clerk integration with middleware protection and layout boundaries.

```typescript
// Middleware-level route protection
const isProtectedRoute = createRouteMatcher([
  '/dashboard(.*)',
  '/:locale/dashboard(.*)'
]);

const isAuthPage = createRouteMatcher([
  '/sign-in(.*)', '/:locale/sign-in(.*)',
  '/sign-up(.*)', '/:locale/sign-up(.*)'
]);
```

**Authentication Flow Pattern**:
```
Middleware Route Check → Conditional Clerk Middleware → Layout Provider → Page Access
```

**Locale-Aware URL Construction**:
```typescript
// Dynamic authentication URL generation
export function getClerkUrls(locale: string) {
  const isDefaultLocale = locale === routing.defaultLocale;
  
  return {
    signInUrl: isDefaultLocale ? '/sign-in' : `/${locale}/sign-in`,
    signUpUrl: isDefaultLocale ? '/sign-up' : `/${locale}/sign-up`, 
    dashboardUrl: isDefaultLocale ? '/dashboard' : `/${locale}/dashboard`,
  };
}
```

### 5. API Route Organization and Patterns

**RESTful API Implementation**: API routes with comprehensive validation and error handling.

```typescript
// API route pattern (/src/app/[locale]/api/counter/route.ts)
export const PUT = async (request: Request) => {
  // 1. Input validation with Zod
  const json = await request.json();
  const parse = CounterValidation.safeParse(json);
  
  if (!parse.success) {
    return NextResponse.json(z.treeifyError(parse.error), { status: 422 });
  }
  
  // 2. Database operation with Drizzle ORM
  const result = await db.insert(counterSchema).values(parse.data).onConflictDoUpdate({
    target: counterSchema.id,
    set: { count: sql`${counterSchema.count} + ${parse.data.count}` },
  }).returning();
  
  // 3. Structured logging
  logger.info('Counter updated', { operation: 'upsert', count: parse.data.count });
  
  return NextResponse.json({ message: 'Success', data: result });
};
```

**API Integration Features**:
- **Type-Safe Validation**: Zod schemas with structured error responses
- **Database Integration**: Drizzle ORM with SQL operations and conflict handling
- **Testing Support**: E2E testing headers for isolated test scenarios
- **Observability**: Structured logging for all API operations

## Key Files and Structure

### Layout Files and Hierarchy

- **`[locale]/layout.tsx`**: Root internationalization layout with global providers
  - **Dependencies**: `next-intl/server`, analytics providers, global CSS
  - **Exports**: Root layout with locale validation and static generation
  - **Pattern**: Async layout with locale validation and provider composition
  - **Features**: Favicon management, metadata configuration, analytics integration

- **`(auth)/layout.tsx`**: Authentication boundary layout with Clerk provider
  - **Dependencies**: `@clerk/nextjs`, locale-aware URL construction
  - **Exports**: Authentication provider with localized configuration
  - **Pattern**: Conditional URL generation based on locale
  - **Features**: Multi-language Clerk UI, locale-aware redirects

- **`(marketing)/layout.tsx`**: Public marketing layout with BaseTemplate integration
  - **Dependencies**: `BaseTemplate`, public navigation components
  - **Exports**: Marketing-focused layout with SEO optimization
  - **Pattern**: Template composition with navigation slots
  - **Features**: Public navigation, marketing-specific styling

### Route Implementation Files

- **Page Components**: Consistent async server component pattern across all routes
  - **Pattern**: `async function Page(props: { params: Promise<{ locale: string }> })`
  - **Features**: Server-side translation loading, metadata generation, static params
  - **Integration**: Internationalization, authentication status awareness

- **API Route Files**: RESTful endpoints with comprehensive error handling
  - **Pattern**: Named exports for HTTP methods (GET, PUT, POST, DELETE)
  - **Features**: Request validation, database integration, structured logging
  - **Integration**: Drizzle ORM, Zod validation, LogTape logging

### Specialized Implementation Files

- **`global-error.tsx`**: Global error boundary with Sentry integration
  - **Pattern**: Client component with error capture and user-friendly fallback
  - **Features**: Sentry error reporting, locale-aware error messages

- **`robots.ts`**: Dynamic robots.txt generation with route exclusions  
  - **Pattern**: Export `robots()` function returning robots configuration
  - **Features**: Protected route exclusions, sitemap reference

- **`sitemap.ts`**: Dynamic sitemap generation with internationalized URLs
  - **Pattern**: Export `sitemap()` function returning URL entries  
  - **Features**: Multi-locale URL generation, automatic discovery

## Integration Points

### Middleware Integration and Request Processing

**Multi-Layer Request Processing**: Sequential middleware execution with performance optimization.

```typescript
// Middleware processing pipeline
export default async function middleware(request: NextRequest, event: NextFetchEvent) {
  // 1. Security layer (Arcjet)
  if (process.env.ARCJET_KEY) {
    const decision = await aj.protect(request);
    if (decision.isDenied()) return NextResponse.json({ error: 'Forbidden' }, { status: 403 });
  }
  
  // 2. Conditional authentication (Clerk)
  if (isAuthPage(request) || isProtectedRoute(request)) {
    return clerkMiddleware(/* locale-aware auth logic */)(request, event);
  }
  
  // 3. Internationalization routing
  return handleI18nRouting(request);
}
```

### Service Layer Integration

**Clean Service Boundaries**: App Router components consume configured services from `/src/libs`.

- **Database Operations**: API routes use `db` from `/src/libs/DB.ts` for type-safe operations  
- **Logging Integration**: Components use `logger` from `/src/libs/Logger.ts` for structured logging
- **Environment Access**: Components access validated environment variables via `Env` from `/src/libs/Env.ts`
- **Authentication**: Layouts integrate with Clerk configuration for protected routes

### Component Integration Patterns

**Server/Client Component Boundaries**: Strategic component placement for optimal performance.

```typescript
// Server components (default)
- All layout components for SEO and performance
- Page components with data fetching and translations  
- BaseTemplate with server-side rendering

// Client components (selective)
- LocaleSwitcher for interactive locale switching
- PostHogProvider for analytics initialization  
- Global error boundary for error handling
- Form components requiring user interaction
```

## Development Patterns

### Static Generation and Performance

**Optimized Static Generation**: Strategic pre-rendering for performance.

```typescript
// Multi-locale static generation for dynamic routes
export function generateStaticParams() {
  return routing.locales
    .map(locale => 
      Array.from({ length: 6 }).map((_, index) => ({ 
        slug: `${index}`, 
        locale 
      }))
    )
    .flat(1);
}

export const dynamicParams = false; // Strict static generation
```

### Metadata and SEO Optimization

**Comprehensive SEO Strategy**: Dynamic metadata generation with internationalization.

```typescript
// Consistent metadata pattern across all pages
export async function generateMetadata(props: PageProps): Promise<Metadata> {
  const { locale } = await props.params;
  const t = await getTranslations({ locale, namespace: 'Page' });
  
  return {
    title: t('meta_title'),
    description: t('meta_description'),
    openGraph: {
      title: t('meta_title'),
      description: t('meta_description'),
    },
  };
}
```

### Error Handling and Monitoring

**Production-Ready Error Handling**: Global error boundaries with observability integration.

- **Global Error Boundary**: Client-side error capture with Sentry reporting
- **API Error Patterns**: Consistent HTTP status codes and structured error responses
- **Middleware Error Handling**: Security and authentication error responses
- **Development Experience**: Enhanced error messages and debugging support

---

*This App Router implementation demonstrates modern Next.js 15 patterns with comprehensive internationalization, authentication integration, and production-ready architecture. For overall source architecture context, see `/src/CONTEXT.md`. For service integration details, see `/src/libs/CONTEXT.md`.*