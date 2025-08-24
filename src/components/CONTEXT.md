# React Component Architecture

*This file documents React component implementation patterns and architecture within the `/src/components` directory.*

## Component Architecture

The `/src/components` directory implements a **modern React 19 component system** with clear server/client boundaries, comprehensive internationalization, analytics integration, and production-ready patterns. The architecture demonstrates enterprise-grade component design with type-safe forms, provider patterns, and accessibility-first development.

### Component Organization Strategy

- **Flat Directory Structure**: Simple organization with minimal nesting for discoverability
- **Functional Grouping**: `/analytics/` subdirectory for specialized analytics components  
- **Server-First Architecture**: Server components as default, client components only when needed
- **Single Responsibility**: Each component handles one specific concern or functionality

### Component Classification

1. **Server Components** (Default): `CurrentCount`, `Hello`, `Sponsors` - Data fetching and static content
2. **Client Components** (`'use client'`): `CounterForm`, `LocaleSwitcher`, analytics components - Interactivity and browser APIs
3. **Provider Components**: `PostHogProvider` - Context management and service integration
4. **Utility Components**: `DemoBadge`, `DemoBanner` - Display and presentation utilities

## Implementation Patterns

### 1. Server/Client Component Decision Pattern

**Server Component Strategy**: Default choice for data fetching, translation loading, and static content.

```typescript
// Server component pattern (CurrentCount.tsx)
export default async function CurrentCount(props: { id: string }) {
  // Server-side data fetching
  const result = await db.query.counterSchema.findMany({
    where: eq(counterSchema.id, props.id),
  });
  
  // Server-side internationalization
  const t = await getTranslations('CurrentCount');
  
  return <span>{t('count', { count: result[0]?.count ?? 0 })}</span>;
}
```

**Client Component Strategy**: Used only for user interaction, browser APIs, and real-time updates.

```typescript
// Client component pattern (CounterForm.tsx)
'use client';

export default function CounterForm(props: { id: string }) {
  const t = useTranslations('CounterForm');
  const router = useRouter();
  
  // Client-side form handling with validation
  const form = useForm({
    resolver: zodResolver(CounterValidation),
    defaultValues: { increment: 0 },
  });
}
```

**Decision Criteria**:
- **Server Component**: Data fetching, translation loading, authentication checks, static content
- **Client Component**: Form handling, user interaction, browser APIs, real-time updates

### 2. Form Handling with Validation Pattern

**Comprehensive Form Architecture**: React Hook Form + Zod validation + internationalization integration.

```typescript
// Complete form handling pattern
import { zodResolver } from '@hookform/resolvers/zod';
import { useForm } from 'react-hook-form';
import { CounterValidation } from '@/validations/CounterValidation';

const form = useForm({
  resolver: zodResolver(CounterValidation),
  defaultValues: { increment: 0 },
});

// Form submission with API integration
async function onSubmit(data: { increment: number }) {
  setIsSubmitting(true);
  try {
    await fetch('/api/counter', {
      method: 'PUT',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ ...data, id: props.id }),
    });
    router.refresh(); // Refresh server components
  } finally {
    setIsSubmitting(false);
  }
}
```

**Form Pattern Features**:
- **Type-Safe Validation**: Zod schemas with TypeScript inference
- **Internationalized Errors**: Error messages through translation keys
- **Loading States**: UI feedback during form submission
- **Server Synchronization**: Router refresh to update server components

### 3. Analytics Integration Pattern

**Provider Architecture**: PostHog integration with performance optimization and privacy controls.

```typescript
// Analytics provider pattern (PostHogProvider.tsx)
'use client';

export default function PostHogProvider({ children }: { children: ReactNode }) {
  useEffect(() => {
    if (typeof window !== 'undefined' && Env.NEXT_PUBLIC_POSTHOG_KEY) {
      posthog.init(Env.NEXT_PUBLIC_POSTHOG_KEY, {
        api_host: Env.NEXT_PUBLIC_POSTHOG_HOST,
        capture_pageview: false, // Manual page view control
        capture_pageleave: true,
      });
    }
  }, []);

  return <PostHogContext.Provider value={posthog}>{children}</PostHogContext.Provider>;
}
```

**Page View Tracking Pattern**:
```typescript
// Optimized page view tracking (PostHogPageView.tsx)
function PostHogPageView() {
  const pathname = usePathname();
  const searchParams = useSearchParams();
  
  useEffect(() => {
    if (pathname) {
      const url = pathname + (searchParams.toString() ? `?${searchParams.toString()}` : '');
      posthog.capture('$pageview', { $current_url: url });
    }
  }, [pathname, searchParams]);

  return null;
}

// Suspense wrapper to prevent SSR issues
export default function SuspendedPostHogPageView() {
  return (
    <Suspense fallback={null}>
      <PostHogPageView />
    </Suspense>
  );
}
```

### 4. Internationalization Component Patterns

**Server-Side Translation Pattern**: For server components with async translation loading.

```typescript
// Server component i18n pattern
export default async function Hello() {
  const t = await getTranslations('Hello');
  
  return (
    <p>
      {t.rich('greeting', {
        name: user?.emailAddresses[0]?.emailAddress ?? '',
        link: (chunks) => <Link href="/about">{chunks}</Link>
      })}
    </p>
  );
}
```

**Client-Side Translation Pattern**: For interactive components with hooks.

```typescript
// Client component i18n pattern
'use client';

export default function LocaleSwitcher() {
  const t = useTranslations('LocaleSwitcher');
  const pathname = usePathname();
  const router = useRouter();
  
  function handleLocaleChange(newLocale: string) {
    router.push(pathname.replace(`/${locale}`, `/${newLocale}`));
    router.refresh();
  }
}
```

**Translation Features**:
- **Rich Text Support**: `t.rich()` for embedded components
- **Parameterized Translations**: Dynamic value insertion
- **Type Safety**: TypeScript integration with translation keys
- **Server/Client Optimization**: Different patterns for different rendering contexts

## Key Files and Structure

### Core Interactive Components

- **`CounterForm.tsx`**: Comprehensive form component with validation and API integration
  - **Dependencies**: React Hook Form, Zod validation, next-intl, Next.js router
  - **Pattern**: Client component with form state management and server synchronization
  - **Features**: Type-safe validation, loading states, internationalized errors

- **`LocaleSwitcher.tsx`**: Internationalization locale switching component
  - **Dependencies**: next-intl navigation hooks, custom routing configuration
  - **Pattern**: Client component with router integration and locale persistence
  - **Features**: Dynamic locale switching, URL rewriting, session persistence

### Data Display Components

- **`CurrentCount.tsx`**: Server-side data fetching and display component
  - **Dependencies**: Drizzle ORM, database connection, internationalization
  - **Pattern**: Server component with async data fetching
  - **Features**: Database integration, E2E testing support, fallback values

- **`Hello.tsx`**: Authentication-aware greeting component
  - **Dependencies**: Clerk authentication, server-side user data access
  - **Pattern**: Server component with conditional authentication display
  - **Features**: User data extraction, rich text translation, authentication fallbacks

### Analytics and Tracking Components

- **`analytics/PostHogProvider.tsx`**: Analytics context provider with privacy controls
  - **Dependencies**: PostHog client library, environment validation
  - **Pattern**: Client component provider with conditional initialization
  - **Features**: Privacy-first analytics, manual page view control, environment-based activation

- **`analytics/PostHogPageView.tsx`**: Page view tracking with performance optimization
  - **Dependencies**: Next.js navigation hooks, PostHog context
  - **Pattern**: Client component with Suspense boundary for SSR compatibility
  - **Features**: URL construction, search param handling, SSR optimization

### Presentation and Utility Components

- **`DemoBadge.tsx`**, **`DemoBanner.tsx`**: Development environment indicators
  - **Pattern**: Simple presentation components with conditional rendering
  - **Features**: Environment detection, responsive styling, accessibility attributes

- **`Sponsors.tsx`**: Static content component with image optimization
  - **Dependencies**: Next.js Image component for performance
  - **Pattern**: Server component with optimized asset loading
  - **Features**: Image optimization, responsive layout, semantic markup

## Integration Points

### Service Layer Integration

**Clean Service Boundaries**: Components consume configured services without implementation details.

- **Database Access**: Server components use `db` from `/src/libs/DB.ts` for type-safe operations
- **Authentication**: Components integrate with Clerk for user data and authentication state
- **Environment Variables**: Type-safe access via `Env` from `/src/libs/Env.ts` with validation
- **Logging**: Components use structured logging patterns for development and production

### App Router Integration

**Routing and Navigation**: Components integrate with Next.js App Router patterns.

```typescript
// Router integration patterns
import { useRouter, usePathname } from 'next/navigation';

// Server component refresh after client actions
router.refresh(); // Updates server components with new data

// Locale-aware navigation
const pathname = usePathname();
router.push(pathname.replace(`/${locale}`, `/${newLocale}`));
```

### Validation and API Integration

**Form-to-API Pipeline**: Components integrate with validation schemas and API endpoints.

- **Client Validation**: Components use Zod schemas for immediate feedback
- **Server Validation**: API routes validate the same schemas for security
- **Error Handling**: Consistent error display and user feedback patterns
- **Type Safety**: End-to-end type safety from component to database

### Authentication and Security

**Security-First Component Design**: Components integrate security patterns throughout.

- **Input Sanitization**: All user inputs validated through Zod schemas
- **Authentication Aware**: Components check user state and handle unauthenticated scenarios  
- **CSRF Protection**: Form submissions include proper headers and validation
- **Environment Security**: Sensitive data accessed only through validated environment variables

## Development Patterns

### Component Testing Strategy

**Testing Architecture**: Components designed for comprehensive testing coverage.

```typescript
// Component testing patterns with i18n support
import { render } from '@testing-library/react';
import { NextIntlClientProvider } from 'next-intl';

// Test wrapper with necessary providers
function renderWithProviders(component: ReactElement) {
  return render(
    <NextIntlClientProvider locale="en" messages={messages}>
      {component}
    </NextIntlClientProvider>
  );
}
```

**Testing Features**:
- **Storybook Integration**: Component stories for visual testing and documentation
- **Vitest Testing**: Unit tests with browser environment support
- **E2E Support**: Components include test identifiers and isolation patterns
- **Provider Testing**: Components testable with necessary context providers

### Accessibility and Performance

**Accessibility-First Design**: Components implement WCAG compliance patterns.

- **Semantic HTML**: Proper heading hierarchy and landmark usage
- **ARIA Labels**: Descriptive labels for interactive elements
- **Focus Management**: Proper focus handling for keyboard navigation
- **Color Contrast**: High contrast colors with accessible styling patterns

**Performance Optimization Patterns**:

- **Server Component Default**: Minimize client-side bundle by defaulting to server rendering
- **Selective Client Components**: Strategic placement of `'use client'` directive
- **Image Optimization**: Next.js Image component for responsive loading
- **Suspense Boundaries**: Prevent SSR compatibility issues with client components

### Component Composition and Reusability

**Composition Strategies**: Components designed for flexible composition and reuse.

```typescript
// Composition pattern example
export interface BaseTemplateProps {
  leftNav?: ReactNode;
  rightNav?: ReactNode;  
  children: ReactNode;
}

// Flexible template composition
<BaseTemplate
  leftNav={<NavigationMenu />}
  rightNav={<UserActions />}
>
  {children}
</BaseTemplate>
```

**Reusability Features**:
- **Configuration Props**: Components accept configuration rather than hard-coding values
- **Children Patterns**: Flexible composition through children and render props
- **Context Integration**: Components integrate with application-wide context providers
- **Environment Adaptation**: Components adapt behavior based on environment variables

---

*This React component architecture demonstrates modern patterns with comprehensive internationalization, analytics integration, and production-ready development practices. For overall source architecture context, see `/src/CONTEXT.md`. For service integration details, see `/src/libs/CONTEXT.md`. For routing integration patterns, see `/src/app/CONTEXT.md`.*