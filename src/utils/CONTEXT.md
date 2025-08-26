# Utility Functions Documentation

## Utility Architecture

**Purpose**: Foundational helper functions and configuration management providing cross-cutting concerns for deployment, internationalization, runtime detection, and application configuration across the Next.js stack.

**Design Philosophy**: Lightweight, pure functions with clear single responsibilities, environment-aware behavior, and comprehensive test coverage supporting the application's core infrastructure needs.

```typescript
// Environment-aware URL generation with deployment fallbacks
export const getBaseUrl = () => {
  if (process.env.NEXT_PUBLIC_APP_URL) return process.env.NEXT_PUBLIC_APP_URL;
  if (process.env.VERCEL_ENV === 'production' && process.env.VERCEL_PROJECT_PRODUCTION_URL) {
    return `https://${process.env.VERCEL_PROJECT_PRODUCTION_URL}`;
  }
  if (process.env.VERCEL_URL) return `https://${process.env.VERCEL_URL}`;
  return 'http://localhost:3000';
};
```

## Implementation Patterns

### Application Configuration Management (`AppConfig.ts`)
```typescript
// Centralized configuration with internationalization support
export const AppConfig = {
  name: 'Nextjs Starter',          // Application branding
  locales: ['en', 'fr'],           // Supported languages
  defaultLocale: 'en',             // Fallback language
  localePrefix: 'as-needed',       // next-intl routing strategy
};

// Clerk authentication localization mapping
export const ClerkLocalizations = {
  defaultLocale: enUS,
  supportedLocales: {
    en: enUS,  // @clerk/localizations
    fr: frFR,
  },
};
```

**Configuration Patterns**:
- **Single Source of Truth**: All application-wide constants in one location
- **Internationalization Ready**: Locale configuration integrated with next-intl and Clerk
- **Type-Safe Exports**: Structured configuration objects with clear type inference
- **Extensible Design**: Easy addition of new locales and configuration properties

### Environment-Aware Deployment Utilities (`Helpers.ts`)
```typescript
// Vercel deployment environment detection with fallbacks
export const getBaseUrl = () => {
  // 1. Explicit configuration override
  if (process.env.NEXT_PUBLIC_APP_URL) {
    return process.env.NEXT_PUBLIC_APP_URL;
  }
  
  // 2. Vercel production environment
  if (process.env.VERCEL_ENV === 'production' && process.env.VERCEL_PROJECT_PRODUCTION_URL) {
    return `https://${process.env.VERCEL_PROJECT_PRODUCTION_URL}`;
  }
  
  // 3. Vercel preview/development
  if (process.env.VERCEL_URL) {
    return `https://${process.env.VERCEL_URL}`;
  }
  
  // 4. Local development fallback
  return 'http://localhost:3000';
};
```

**Deployment Patterns**:
- **Environment Cascading**: Priority-based environment variable resolution
- **Vercel Integration**: Native support for Vercel deployment environments
- **HTTPS by Default**: Automatic HTTPS for non-localhost environments
- **Development Safety**: Fallback to localhost for local development

### Internationalization Path Utilities
```typescript
// Locale-aware URL path construction
export const getI18nPath = (url: string, locale: string) => {
  if (locale === routing.defaultLocale) {
    return url;  // Default locale URLs have no prefix
  }
  return `/${locale}${url}`;  // Non-default locales prefixed
};
```

**I18n Patterns**:
- **Default Locale Optimization**: Clean URLs for primary language (no `/en` prefix)
- **Consistent Routing**: Integration with next-intl routing configuration
- **Path Construction**: Safe URL building with locale awareness
- **SEO-Friendly**: URL structure optimized for search engines and user experience

### Runtime Environment Detection
```typescript
// Server-side rendering detection utility
export const isServer = () => {
  return typeof window === 'undefined';
};
```

**Runtime Patterns**:
- **SSR/CSR Detection**: Reliable server-side rendering identification
- **Hydration Safety**: Prevents client-server mismatch during hydration
- **Conditional Logic**: Enables environment-specific code paths
- **Performance Optimization**: Allows server-only or client-only operations

### Database Connection Factory (`DBConnection.ts`)
```typescript
// Reusable database connection factory with connection pooling
export const createDbConnection = () => {
  const pool = new Pool({
    connectionString: Env.DATABASE_URL,
    ssl: !Env.DATABASE_URL.includes('localhost') && !Env.DATABASE_URL.includes('127.0.0.1'),
    max: 1,
  });

  return drizzle({ client: pool, schema });
};
```

**Connection Factory Patterns**:
- **Modular Architecture**: Separated connection logic allows reuse across different contexts
- **SSL Auto-Detection**: Intelligent SSL configuration based on connection string patterns
- **Connection Pooling**: PostgreSQL connection pool with resource control (max: 1)
- **Environment Integration**: Uses validated environment variables from `Env.ts`
- **Schema Integration**: Type-safe database operations with automatic schema binding

### Database Migration Automation (`DBMigration.ts`)
```typescript
// Automatic migration execution with dedicated connection management
try {
  const migrationDb = createDbConnection();
  await migrate(migrationDb, {
    migrationsFolder: path.join(process.cwd(), 'migrations'),
  });
} finally {
  await migrationDb.end?.();
}
```

**Migration Patterns**:
- **Dedicated Connections**: Uses DBConnection factory for controlled migration execution
- **Initialization Hook**: Automatic execution via Next.js `instrumentation.ts`
- **Filesystem Integration**: Direct migration file discovery from `/migrations`
- **Connection Lifecycle**: Proper connection cleanup with try-finally blocks
- **Development Workflow**: Seamless migration application on server restart
- **Production Safety**: Controlled migration execution in deployment environments

## Key Files and Structure

### Utility Organization
```
src/utils/
├── AppConfig.ts       # Application configuration and internationalization settings
├── Helpers.ts         # Environment, URL, and runtime utilities
├── Helpers.test.ts    # Unit tests for helper functions
├── DBConnection.ts    # Database connection factory with connection pooling
└── DBMigration.ts     # Database migration automation utilities
```

**File Responsibilities**:
- **AppConfig.ts**: Centralized application constants, branding, and localization configuration
- **Helpers.ts**: Environment detection, URL generation, i18n path construction, runtime utilities
- **DBConnection.ts**: Reusable database connection factory with SSL auto-detection and connection pooling
- **DBMigration.ts**: Database schema migration automation via Drizzle ORM using DBConnection factory
- **Helpers.test.ts**: Comprehensive test coverage for utility function behavior

### Function Usage Distribution
```typescript
// AppConfig usage across components
import { AppConfig } from '@/utils/AppConfig';           // Templates, routing
import { ClerkLocalizations } from '@/utils/AppConfig';  // Authentication layouts

// Database connection factory usage
import { createDbConnection } from '@/utils/DBConnection';  // DB.ts, DBMigration.ts, testing

// Helper function usage across stack
import { getBaseUrl } from '@/utils/Helpers';      // Sitemap, robots.txt
import { getI18nPath } from '@/utils/Helpers';     // Auth pages, navigation
import { isServer } from '@/utils/Helpers';        // Conditional rendering
```

## Integration Points

### Template and Layout Integration (`/src/templates/BaseTemplate.tsx`)
```typescript
// Application branding and metadata
import { AppConfig } from '@/utils/AppConfig';

<title>{AppConfig.name}</title>
<footer>
  © Copyright {new Date().getFullYear()} {AppConfig.name}
</footer>
```

**Template Patterns**:
- **Branding Consistency**: Centralized application name across all templates
- **Dynamic Copyright**: Automatic year updates using configuration
- **Metadata Integration**: SEO title generation from configuration
- **Maintainability**: Single location updates propagate across entire application

### Internationalization System Integration (`/src/libs/I18nRouting.ts`)
```typescript
// Routing configuration derived from AppConfig
import { AppConfig } from '@/utils/AppConfig';

export const routing = createNavigation({
  locales: AppConfig.locales,
  defaultLocale: AppConfig.defaultLocale,
  localePrefix: AppConfig.localePrefix,
});
```

**Routing Integration Patterns**:
- **Configuration Consistency**: Single source of truth for locale settings
- **next-intl Integration**: Direct configuration injection into routing system
- **Type Safety**: Configuration changes automatically update routing types
- **Centralized Management**: Locale changes require only AppConfig updates

### Authentication Layout Integration (`/src/app/[locale]/(auth)/layout.tsx`)
```typescript
// Clerk localization from AppConfig
import { ClerkLocalizations } from '@/utils/AppConfig';

const clerkLocale = ClerkLocalizations.supportedLocales[locale] 
  ?? ClerkLocalizations.defaultLocale;

<ClerkProvider localization={clerkLocale}>
```

**Authentication Patterns**:
- **Locale Consistency**: Unified localization across app and authentication
- **Fallback Handling**: Graceful degradation for unsupported locales
- **Provider Configuration**: Dynamic Clerk localization based on user locale
- **Type Safety**: Localization mapping provides compile-time validation

### SEO and Meta Integration (`/src/app/sitemap.ts`, `/src/app/robots.ts`)
```typescript
// Environment-aware URL generation for SEO
import { getBaseUrl } from '@/utils/Helpers';

export default function sitemap(): MetadataRoute.Sitemap {
  return [{
    url: getBaseUrl(),
    lastModified: new Date(),
    changeFrequency: 'yearly',
    priority: 1,
  }];
}
```

**SEO Integration Patterns**:
- **Dynamic URL Generation**: Environment-specific sitemap and robots.txt URLs
- **Deployment Consistency**: Same base URL logic across development and production
- **Search Engine Optimization**: Proper canonical URL generation
- **Environment Safety**: Automatic HTTPS/HTTP protocol detection

### Navigation and Routing Integration
```typescript
// Locale-aware navigation paths
import { getI18nPath } from '@/utils/Helpers';

// In auth pages and navigation components
const signInUrl = getI18nPath('/sign-in', locale);
const dashboardUrl = getI18nPath('/dashboard', locale);
```

**Navigation Patterns**:
- **Consistent URL Construction**: Standardized locale prefix handling
- **SEO-Friendly URLs**: Clean URLs for default locale, prefixed for others
- **Route Safety**: Guaranteed proper locale handling across navigation
- **Link Generation**: Automated internationalized link construction

## Development Patterns

### Testing Strategy (`Helpers.test.ts`)
```typescript
// Comprehensive test coverage for utility functions
describe('getI18nPath function', () => {
  it('should not change the path for default language', () => {
    const url = '/random-url';
    const locale = routing.defaultLocale;
    expect(getI18nPath(url, locale)).toBe(url);
  });

  it('should prepend the locale to the path for non-default language', () => {
    const url = '/random-url';
    const locale = 'fr';
    expect(getI18nPath(url, locale)).toMatch(/^\/fr/);
  });
});
```

**Testing Patterns**:
- **Behavior-Driven Testing**: Tests focus on expected behavior rather than implementation
- **Integration Testing**: Tests use actual routing configuration for realistic scenarios
- **Edge Case Coverage**: Default locale handling and non-default locale prefixing
- **Regression Prevention**: Automated tests prevent breaking changes to utility functions

### Configuration Evolution Workflow
1. **Update AppConfig**: Modify locales, application name, or routing strategy
2. **TypeScript Validation**: Configuration changes surface type errors in consuming code
3. **Test Integration**: Run utility tests to verify configuration consistency
4. **Component Updates**: Update templates, layouts, and routing as needed
5. **Deployment Verification**: Test environment-specific behavior across deployment stages

### Environment Configuration Management
```typescript
// Development vs Production behavior
const baseUrl = getBaseUrl();
// Development: 'http://localhost:3000'
// Vercel Preview: 'https://app-branch-user.vercel.app'
// Production: 'https://yourdomain.com' or Vercel production URL
```

**Environment Patterns**:
- **Graceful Defaults**: Safe fallbacks for missing environment variables
- **Development Experience**: Zero-configuration local development
- **Production Safety**: Explicit configuration options for production deployments
- **Vercel Integration**: Native support for Vercel deployment environments

### Migration Integration (`instrumentation.ts`)
```typescript
// Automatic database migration on server startup
export async function register() {
  if (process.env.NEXT_RUNTIME === 'nodejs') {
    const { logger } = await import('./libs/Logger');
    logger.info('Server instrumentation started');
    
    // Automatic migration execution
    await import('./utils/DBMigration');
  }
}
```

**Migration Integration Patterns**:
- **Server-Only Execution**: Migrations run only in Node.js runtime environment
- **Startup Integration**: Database schema updates occur before application serves requests
- **Zero-Downtime Strategy**: Migrations execute during application initialization
- **Development Workflow**: Schema changes automatically applied on server restart

## System Integration Context

### Cross-Component Dependencies
- **Templates** (`/src/templates/`): Application branding and metadata from AppConfig
- **Routing System** (`/src/libs/I18nRouting.ts`): Locale configuration and routing behavior
- **Authentication** (`/src/app/[locale]/(auth)/`): Clerk localization and auth page routing
- **SEO Components** (`/src/app/sitemap.ts`, `/src/app/robots.ts`): Environment-aware URL generation
- **Database Layer** (`instrumentation.ts`): Automatic migration execution
- **Component System**: Runtime detection for conditional rendering

### External Service Integration
- **next-intl**: Locale configuration and routing integration
- **Clerk Authentication**: Localization mapping and authentication page routing
- **Vercel Platform**: Environment detection and deployment-specific URL generation
- **Drizzle ORM**: Database migration automation and schema management

### Future Expansion Patterns
- **New Configuration Properties**: Extend AppConfig with additional application constants
- **Additional Locales**: Add new languages to locales array and localization mappings
- **Environment Variables**: Extend getBaseUrl() with new deployment environment support
- **Utility Functions**: Add new helper functions following established patterns (pure functions, single responsibility)
- **Testing Coverage**: Expand test suite for new utilities and edge cases

This utility system provides the foundational infrastructure for configuration management, environment detection, internationalization, and deployment across the entire Next.js application stack while maintaining clear separation of concerns and comprehensive test coverage.