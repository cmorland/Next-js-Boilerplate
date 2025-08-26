# NextJS-BP-SaaS Project Structure

This document provides the complete technology stack and file tree structure for the NextJS-BP-SaaS boilerplate project. **AI agents MUST read this file to understand the project organization before making any changes.**

## Technology Stack

### Core Frontend Technologies
- **TypeScript 5.8.3** with **npm** - Static typing and dependency management
- **Next.js 15.5.0** - Full-stack React framework with App Router, API routes, and internationalization
- **React 19.1.1** - Modern React with latest features and server components
- **@t3-oss/env-nextjs 0.13.8** - Type-safe environment variable validation with Zod

### Styling & UI Framework
- **Tailwind CSS 4.1.12** - Utility-first CSS framework with PostCSS integration
- **PostCSS 8.5.6** - CSS processing with Tailwind CSS plugin
- **React Hook Form 7.62.0** - Performant form handling with minimal re-renders
- **@hookform/resolvers 5.2.1** - Validation resolvers with Zod integration

### Database & Backend Services
- **Drizzle ORM 0.44.4** - TypeScript-first ORM for SQL databases
- **PostgreSQL** with **PGLite 0.0.12** - Production PostgreSQL with lightweight development database
- **Drizzle Kit 0.31.4** - Database migration and introspection tools

### Authentication & Security
- **Clerk (@clerk/nextjs 6.31.3)** - Complete authentication solution with localization support
- **Arcjet 1.0.0-beta.10** - Security and bot protection with rate limiting
- **Zod 4.0.17** - TypeScript-first schema validation for forms and API validation

### Integration Services & APIs
- **Sentry 9.46.0** - Error tracking and performance monitoring with source maps
- **PostHog 1.260.1** - Product analytics and feature flags with privacy-first approach  
- **next-intl 4.3.4** - Internationalization with server-side rendering optimization
- **@logtape/logtape 1.0.4** - Structured logging for development and production

### Development & Quality Tools
- **ESLint 9.33.0** with **@antfu/eslint-config 4.19.0** - Code quality and formatting (no Prettier)
- **TypeScript 5.8.3** - Static type checking with strict configuration
- **Vitest 3.2.4** - Unit testing framework with browser and Node.js environments
- **Playwright 1.54.2** - End-to-end testing with visual regression testing
- **Storybook 9.1.2** - Component development and documentation with accessibility testing

### Build & Deployment Tools
- **Turbopack** - Fast development builds integrated with Next.js
- **@next/bundle-analyzer** - Bundle size analysis and optimization
- **Semantic Release 24.2.7** - Automated versioning and releases
- **Lefthook 1.12.3** - Git hooks management with pre-commit quality checks

### Monitoring & Observability
- **Checkly 6.4.0** - API monitoring and synthetic E2E testing in production
- **Codecov** - Code coverage reporting and analysis
- **@spotlightjs/spotlight 3.0.2** - Development debugging and performance analysis
- **Crowdin** - Translation management with automated workflow integration

### Future Technologies
- **Claude AI Integration** - AI-first development with custom commands and automation hooks
- **Chromatic** - Visual regression testing for Storybook components
- **Better Stack** - Centralized logging infrastructure (configured but optional)

## Complete NextJS-BP-SaaS Project Structure

```
NextJS-BP-SaaS/
├── README.md                                    # Project overview and setup instructions
├── CLAUDE.md                                    # Master AI context file with project guidelines
├── MCP-ASSISTANT-RULES.md                       # AI assistant rules and conventions
├── LICENSE                                      # MIT License
├── package.json                                 # Node.js dependencies and build scripts
├── package-lock.json                            # Dependency lock file for reproducible installs
├── tsconfig.json                                # TypeScript configuration with strict settings
├── next.config.ts                               # Next.js configuration with i18n and Sentry
├── eslint.config.mjs                            # ESLint configuration using @antfu/eslint-config
├── vitest.config.mts                            # Vitest testing configuration (unit + UI tests)
├── playwright.config.ts                         # Playwright E2E testing configuration
├── postcss.config.mjs                           # PostCSS configuration for Tailwind CSS
├── commitlint.config.ts                         # Conventional commits enforcement
├── knip.config.ts                               # Dependency analysis and dead code detection
├── lefthook.yml                                 # Git hooks for pre-commit quality checks
├── drizzle.config.ts                            # Database ORM configuration for PostgreSQL
├── checkly.config.ts                            # API monitoring and synthetic testing
├── codecov.yml                                  # Code coverage reporting configuration
├── crowdin.yml                                  # Translation management workflow
├── .env                                         # Environment variables (development)
├── .env.production                              # Production environment configuration
├── .gitignore                                   # Git ignore patterns for Node.js and Next.js
├── .coderabbit.yaml                             # AI-powered code review configuration
│
├── .github/                                     # GitHub Actions and repository configuration
│   ├── FUNDING.yml                              # GitHub Sponsors configuration
│   ├── dependabot.yml                           # Automated dependency updates
│   ├── actions/
│   │   └── setup-project/
│   │       └── action.yml                       # Reusable GitHub Actions setup
│   └── workflows/                               # CI/CD automation workflows
│       ├── CI.yml                               # Main CI pipeline (build, test, deploy)
│       ├── checkly.yml                          # Post-deployment API monitoring
│       ├── crowdin.yml                          # Translation synchronization
│       └── release.yml                          # Automated semantic releases
│
├── .claude/                                     # Claude AI Development Kit integration
│   ├── settings.local.json                      # Claude-specific configuration
│   ├── commands/                                # Custom Claude commands for development
│   │   ├── README.md                            # Commands documentation
│   │   ├── code-review.md                       # Automated code review workflows
│   │   ├── create-docs.md                       # Documentation generation
│   │   ├── full-context.md                      # Full codebase analysis
│   │   ├── gemini-consult.md                    # Gemini AI consultation integration
│   │   ├── handoff.md                           # Task handoff between AI agents
│   │   ├── refactor.md                          # Code refactoring assistance
│   │   └── update-docs.md                       # Documentation maintenance
│   └── hooks/                                   # Automation hooks for AI workflows
│       ├── README.md                            # Hooks system documentation
│       ├── config/
│       │   └── sensitive-patterns.json          # Security pattern detection
│       ├── setup/                               # Environment setup automation
│       ├── sounds/                              # Notification sounds for workflows
│       ├── gemini-context-injector.sh           # Gemini AI context injection
│       ├── mcp-security-scan.sh                 # Security vulnerability scanning
│       ├── notify.sh                            # Notification system for workflows
│       └── subagent-context-injector.sh         # Context injection for sub-agents
│
├── .storybook/                                  # Storybook component development environment
│   ├── main.ts                                  # Storybook main configuration with Next.js
│   ├── preview.ts                               # Global decorators and parameters
│   ├── vitest.config.mts                        # Storybook component testing
│   └── vitest.setup.ts                          # Test environment setup
│
├── .vscode/                                     # VS Code workspace configuration
│   ├── extensions.json                          # Recommended VS Code extensions
│   ├── launch.json                              # Debug configurations for Next.js
│   ├── settings.json                            # Workspace settings (ESLint, i18n, TypeScript)
│   └── tasks.json                               # Build and development tasks
│
├── docs/                                        # Project documentation and specifications
│   ├── README.md                                # Documentation overview and guidelines
│   ├── CONTEXT-tier2-component.md               # Component-level context template
│   ├── CONTEXT-tier3-feature.md                 # Feature-level context template
│   ├── ai-context/                              # AI-specific project documentation
│   │   ├── deployment-infrastructure.md         # Infrastructure and deployment guidance
│   │   ├── docs-overview.md                     # Documentation architecture overview
│   │   ├── handoff.md                           # Task management and handoff protocols
│   │   ├── project-structure.md                 # This file - complete project structure
│   │   └── system-integration.md                # External system integration patterns
│   ├── open-issues/                             # Issue tracking and problem documentation
│   │   └── example-api-performance-issue.md     # Issue template and examples
│   └── specs/                                   # Technical specifications and requirements
│       ├── example-api-integration-spec.md      # API integration specification template
│       └── example-feature-specification.md     # Feature specification template
│
├── public/                                      # Static assets served by Next.js
│   ├── favicon.ico                              # Website favicon (16x16, 32x32, ICO format)
│   ├── favicon-16x16.png                        # PNG favicon variants
│   ├── favicon-32x32.png                        # PNG favicon variants
│   ├── apple-touch-icon.png                     # Apple device home screen icon
│   └── assets/
│       └── images/                              # Project images, logos, and screenshots
│           ├── arcjet-dark.svg                  # Technology partner logos (dark theme)
│           ├── arcjet-light.svg                 # Technology partner logos (light theme)
│           ├── nextjs-boilerplate-saas.png      # Project screenshots for documentation
│           ├── nextjs-boilerplate-sign-in.png   # Authentication flow screenshots
│           └── [various service logos]          # Partner and service provider logos
│
├── src/                                         # Application source code
│   ├── app/                                     # Next.js App Router (file-based routing)
│   │   ├── [locale]/                            # Internationalized route structure
│   │   │   ├── layout.tsx                       # Root locale layout with providers
│   │   │   ├── (auth)/                          # Authentication route group
│   │   │   │   ├── layout.tsx                   # Auth-specific layout with guards
│   │   │   │   ├── (center)/                    # Centered authentication pages
│   │   │   │   │   ├── layout.tsx               # Centered layout wrapper
│   │   │   │   │   ├── sign-in/[[...sign-in]]/
│   │   │   │   │   │   └── page.tsx             # Clerk sign-in page integration
│   │   │   │   │   └── sign-up/[[...sign-up]]/
│   │   │   │   │       └── page.tsx             # Clerk sign-up page integration
│   │   │   │   └── dashboard/                   # Protected dashboard routes
│   │   │   │       ├── layout.tsx               # Dashboard layout with navigation
│   │   │   │       ├── page.tsx                 # Main dashboard page
│   │   │   │       └── user-profile/[[...user-profile]]/
│   │   │   │           └── page.tsx             # Clerk user profile management
│   │   │   ├── (marketing)/                     # Public marketing route group
│   │   │   │   ├── layout.tsx                   # Marketing layout with header/footer
│   │   │   │   ├── page.tsx                     # Home page with hero and features
│   │   │   │   ├── about/page.tsx               # About page
│   │   │   │   ├── counter/page.tsx             # Interactive counter demo
│   │   │   │   └── portfolio/                   # Dynamic portfolio section
│   │   │   │       ├── page.tsx                 # Portfolio listing page
│   │   │   │       └── [slug]/page.tsx          # Dynamic portfolio item pages
│   │   │   └── api/                             # Next.js API routes
│   │   │       └── counter/route.ts             # RESTful counter API endpoint
│   │   ├── global-error.tsx                     # Global error boundary with Sentry
│   │   ├── robots.ts                            # Dynamic robots.txt generation
│   │   └── sitemap.ts                           # Dynamic XML sitemap generation
│   │
│   ├── components/                              # Reusable React components
│   │   ├── CounterForm.tsx                      # Interactive counter form with validation
│   │   ├── CurrentCount.tsx                     # Real-time counter display component
│   │   ├── DemoBadge.tsx                        # Demo/prototype indicator badge
│   │   ├── DemoBanner.tsx                       # Promotional banner component
│   │   ├── Hello.tsx                            # Internationalized greeting component
│   │   ├── LocaleSwitcher.tsx                   # Language selection dropdown
│   │   ├── Sponsors.tsx                         # Sponsor/partner logo display
│   │   └── analytics/                           # Analytics and tracking components
│   │       ├── PostHogPageView.tsx              # Page view tracking with PostHog
│   │       └── PostHogProvider.tsx              # Analytics context provider
│   │
│   ├── libs/                                    # Library configurations and integrations
│   │   ├── Arcjet.ts                            # Security middleware (rate limiting, bot protection)
│   │   ├── DB.ts                                # Database connection management (Drizzle + PostgreSQL)
│   │   ├── Env.ts                               # Type-safe environment variable validation
│   │   ├── I18n.ts                              # Internationalization configuration (next-intl)
│   │   ├── I18nNavigation.ts                    # Internationalized navigation helpers
│   │   ├── I18nRouting.ts                       # Locale-based routing configuration
│   │   └── Logger.ts                            # Structured logging setup (LogTape)
│   │
│   ├── locales/                                 # Internationalization translation files
│   │   ├── en.json                              # English translations (default locale)
│   │   └── fr.json                              # French translations
│   │
│   ├── models/                                  # Database schema and data models
│   │   └── Schema.ts                            # Drizzle ORM schema definitions (PostgreSQL)
│   │
│   ├── styles/                                  # Global styling and CSS configuration
│   │   └── global.css                           # Tailwind CSS imports and global styles
│   │
│   ├── templates/                               # Page layout templates with testing
│   │   ├── BaseTemplate.tsx                     # Base page template with common layout
│   │   ├── BaseTemplate.test.tsx                # Template component tests (Vitest)
│   │   └── BaseTemplate.stories.tsx             # Storybook component stories
│   │
│   ├── types/                                   # TypeScript type definitions
│   │   └── I18n.ts                              # Internationalization type definitions
│   │
│   ├── utils/                                   # Utility functions and helpers
│   │   ├── AppConfig.ts                         # Application-wide configuration constants
│   │   ├── DBConnection.ts                      # Database connection factory with connection pooling
│   │   ├── DBMigration.ts                       # Database migration utilities
│   │   ├── Helpers.ts                           # General-purpose helper functions
│   │   └── Helpers.test.ts                      # Helper function unit tests
│   │
│   ├── validations/                             # Input validation schemas (Zod)
│   │   └── CounterValidation.ts                 # Counter API validation rules
│   │
│   ├── middleware.ts                            # Next.js middleware (auth, i18n, security)
│   ├── instrumentation.ts                       # Application instrumentation (Sentry)
│   └── instrumentation-client.ts                # Client-side instrumentation setup
│
├── migrations/                                  # Database migration files (Drizzle)
│   ├── 0000_init-db.sql                         # Initial database schema
│   └── meta/                                    # Migration metadata and versioning
│       ├── 0000_snapshot.json                   # Schema snapshot for consistency
│       └── _journal.json                        # Migration execution journal
│
├── tests/                                       # Comprehensive test suites
│   ├── e2e/                                     # End-to-end tests (Playwright)
│   │   ├── Counter.e2e.ts                       # Counter functionality E2E tests
│   │   ├── I18n.e2e.ts                          # Internationalization E2E tests
│   │   ├── Sanity.check.e2e.ts                  # Basic functionality smoke tests
│   │   └── Visual.e2e.ts                        # Visual regression tests (Chromatic)
│   └── integration/                             # Integration tests (Vitest)
│       └── Counter.spec.ts                      # Counter API integration tests
│
└── logs/                                        # Application logs (development)
```

## Key Architectural Patterns

### Route Groups & Organization
- **Next.js App Router**: Uses route groups `(auth)`, `(marketing)`, `(center)` for logical organization without affecting URLs
- **Internationalization**: Built-in i18n support with `[locale]` dynamic routing and translation files
- **API Integration**: Centralized API routes with RESTful patterns and type-safe validation

### Development Workflow
- **AI-First Development**: Extensive Claude AI integration with custom commands, hooks, and context injection
- **Type Safety**: Comprehensive TypeScript configuration with strict type checking across all layers
- **Quality Gates**: Multiple layers of quality assurance including ESLint, TypeScript checking, testing, and automated CI/CD
- **Testing Strategy**: Multi-environment testing (unit, integration, E2E, visual regression) with coverage reporting

### Security & Monitoring
- **Defense in Depth**: Multiple security layers with Arcjet, Clerk authentication, and environment validation
- **Observability**: Comprehensive monitoring with Sentry, PostHog analytics, and Checkly synthetic testing
- **Compliance**: GDPR-ready with privacy-first analytics and secure authentication patterns

---

*This NextJS-BP-SaaS project structure represents a production-ready, AI-assisted development environment with enterprise-grade features including authentication, internationalization, comprehensive testing, monitoring, and modern development practices. All AI agents should reference this document before making changes to understand the complete technology stack and organizational patterns.*