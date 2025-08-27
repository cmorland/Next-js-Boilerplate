# Documentation Architecture

This project uses a **3-tier documentation system** that organizes knowledge by stability and scope, enabling efficient AI context loading and scalable development.

## How the 3-Tier System Works

**Tier 1 (Foundation)**: Stable, system-wide documentation that rarely changes - architectural principles, technology decisions, cross-component patterns, and core development protocols.

**Tier 2 (Component)**: Architectural charters for major components - high-level design principles, integration patterns, and component-wide conventions without feature-specific details.

**Tier 3 (Feature-Specific)**: Granular documentation co-located with code - specific implementation patterns, technical details, and local architectural decisions that evolve with features.

This hierarchy allows AI agents to load targeted context efficiently while maintaining a stable foundation of core knowledge.

## Documentation Principles
- **Co-location**: Documentation lives near relevant code
- **Smart Extension**: New documentation files created automatically when warranted
- **AI-First**: Optimized for efficient AI context loading and machine-readable patterns

## Tier 1: Foundational Documentation (System-Wide)

- **[Master Context](/CLAUDE.md)** - *Essential for every session.* Coding standards, security requirements, MCP server integration patterns, and development protocols
- **[Project Structure](/docs/ai-context/project-structure.md)** - *REQUIRED reading.* Complete technology stack, file tree, and system architecture. Must be attached to Gemini consultations
- **[System Integration](/docs/ai-context/system-integration.md)** - *For cross-component work.* Communication patterns, data flow, testing strategies, and performance optimization
- **[Deployment Infrastructure](/docs/ai-context/deployment-infrastructure.md)** - *Infrastructure patterns.* Containerization, monitoring, CI/CD workflows, and scaling strategies
- **[Task Management](/docs/ai-context/handoff.md)** - *Session continuity.* Current tasks, documentation system progress, and next session goals

## Tier 2: Component-Level Documentation

### NextJS-BP-SaaS Components
- **[Source Code Architecture](/src/CONTEXT.md)** - *Core implementation.* Next.js 15 App Router patterns, TypeScript architecture, security middleware, internationalization, and service integration patterns

## Tier 3: Feature-Specific Documentation

Granular CONTEXT.md files co-located with code for minimal cascade effects:

### NextJS-BP-SaaS Feature Documentation
*Granular implementation documentation co-located with code:*

- **[Service Integration](/src/libs/CONTEXT.md)** - *External services.* Authentication (Clerk), security (Arcjet), analytics (PostHog, Sentry), database (Drizzle), and internationalization integration patterns with production-ready configuration management
- **[App Router Patterns](/src/app/CONTEXT.md)** - *Routing architecture.* Next.js 15 App Router implementation with route groups, layout composition, internationalization routing, authentication flows, and API route patterns
- **[Component System](/src/components/CONTEXT.md)** - *UI patterns.* React 19 component architecture with server/client boundaries, form handling (React Hook Form + Zod), analytics integration (PostHog), internationalization patterns, and accessibility-first design
- **[Database Schema](/src/models/CONTEXT.md)** - *Data patterns.* Drizzle ORM PostgreSQL schemas with centralized schema definition, automatic TypeScript inference, migration management, type-safe database operations across Next.js stack, and evolution-friendly design patterns
- **[Input Validation](/src/validations/CONTEXT.md)** - *Validation patterns.* Zod 3.x schemas with TypeScript integration, cross-stack validation consistency (client forms + API routes), business rule enforcement, structured error handling, and environment configuration validation
- **[Utility Functions](/src/utils/CONTEXT.md)** - *Helper patterns.* Environment-aware deployment utilities, internationalization path construction, application configuration management, runtime detection, database migration automation, and cross-cutting infrastructure functions with comprehensive test coverage

## Adding New Documentation

### New Component
1. Create `/new-component/CONTEXT.md` (Tier 2)
2. Add entry to this file under appropriate section
3. Create feature-specific Tier 3 docs as features develop

### New Feature
1. Create `/component/src/feature/CONTEXT.md` (Tier 3)
2. Reference parent component patterns
3. Add entry to this file under component's features

### Deprecating Documentation
1. Remove obsolete CONTEXT.md files
2. Update this mapping document
3. Check for broken references in other docs

---

*This documentation architecture template should be customized to match your project's actual structure and components. Add or remove sections based on your architecture.*
