# NextJS-BP-SaaS - AI Context (claude-master)

## 1. Project Overview
- **Vision:** Production-ready Next.js 15 SaaS boilerplate with enterprise-grade authentication, security, internationalization, and comprehensive AI-assisted development integration
- **Current Phase:** Production Ready with Comprehensive Documentation System
- **Key Architecture:** Next.js 15 + App Router + React 19 + TypeScript 5.8.3 + Clerk Auth + Arcjet Security + Drizzle ORM + PostgreSQL + PostHog Analytics + Sentry Monitoring + next-intl + Tailwind CSS + Claude Code Integration
- **Development Strategy:** AI-First Development with Systematic Documentation, Type Safety and Automated Quality Gates

## 2. Project Structure

**⚠️ CRITICAL: AI agents MUST read the [Project Structure documentation](/docs/ai-context/project-structure.md) before attempting any task to understand the complete technology stack, file tree and project organization.**

NextJS-BP-SaaS is a modern full-stack boilerplate with an AI-Powered development workflow. For the complete tech stack and file tree structure, see [docs/ai-context/project-structure.md](/docs/ai-context/project-structure.md).

## 3. Coding Standards & AI Instructions

### General Instructions
- Your most important job is to manage your own context. Always read any relevant files BEFORE planning changes.
- When updating documentation, keep updates concise and on point to prevent bloat.
- Write code following KISS, YAGNI, and DRY principles.
- When in doubt follow proven best practices for implementation.
- Do not commit to git without user approval.
- Do not run any servers, rather tell the user to run servers for testing.
- Always consider industry standard libraries/frameworks first over custom implementations.
- Never mock anything. Never use placeholders. Never omit code.
- Apply SOLID principles where relevant. Use modern framework features rather than reinventing solutions.
- Be brutally honest about whether an idea is good or bad.
- Make side effects explicit and minimal.
- Design database schema to be evolution-friendly (avoid breaking changes).


### File Organization & Modularity
- Default to creating multiple small, focused files rather than large monolithic ones
- Each file should have a single responsibility and clear purpose
- Keep files under 350 lines when possible - split larger files by extracting utilities, constants, types, or logical components into separate modules
- Separate concerns: utilities, constants, types, components, and business logic into different files
- Prefer composition over inheritance - use inheritance only for true 'is-a' relationships, favor composition for 'has-a' or behavior mixing

- Follow existing project structure and conventions - place files in appropriate directories. Create new directories and move files if deemed appropriate.
- Use well defined sub-directories to keep things organized and scalable
- Structure projects with clear folder hierarchies and consistent naming conventions
- Import/export properly - design for reusability and maintainability

### TypeScript & Type Safety (REQUIRED)
- **Always** use TypeScript interfaces for component props and function parameters
- **Strict Type Checking**: All code must pass TypeScript strict mode
- **Runtime Validation**: Use Zod schemas for API validation and environment variables
- **Database Types**: Leverage Drizzle ORM automatic type inference
- **Component Types**: Proper React component typing with server/client boundaries

```typescript
// Good - API Route with end-to-end type safety
export const PUT = async (request: Request): Promise<NextResponse> => {
  const json = await request.json();
  const parse = CounterValidation.safeParse(json);
  
  if (!parse.success) {
    return NextResponse.json(z.treeifyError(parse.error), { status: 422 });
  }
  
  const result = await db.insert(counterSchema).values(parse.data);
  return NextResponse.json(result);
};
```

### Naming Conventions
- **Components**: PascalCase (e.g., `CounterForm`, `BaseTemplate`)
- **Functions/Utilities**: camelCase (e.g., `getBaseUrl`, `getI18nPath`)
- **Constants/Config**: PascalCase objects (e.g., `AppConfig`, `ClerkLocalizations`)
- **Types/Interfaces**: PascalCase with descriptive suffixes (e.g., `CounterValidationType`)
- **Files**: PascalCase for components, camelCase for utilities, lowercase for pages


### Documentation Requirements
- Every component needs comprehensive JSDoc comments
- Every utility function needs parameter and return type documentation
- Use TSDoc standard with @param and @returns tags
- Maintain CONTEXT.md files for architectural documentation
- Follow 3-tier documentation system (Foundation → Component → Feature-specific)

```typescript
/**
 * Generate environment-aware base URL for the application
 * 
 * @returns The appropriate base URL based on deployment environment
 * Priority: NEXT_PUBLIC_APP_URL → Vercel production → Vercel preview → localhost
 */
export const getBaseUrl = (): string => {
  if (process.env.NEXT_PUBLIC_APP_URL) {
    return process.env.NEXT_PUBLIC_APP_URL;
  }
  // Additional environment detection logic...
};
```

### Security First
- **Defense in Depth**: Multi-layer security with Arcjet → Clerk → API validation pipeline
- **Environment Validation**: All secrets validated with Zod schemas using @t3-oss/env-nextjs
- **Input Sanitization**: Validate all API inputs with Zod schemas before database operations
- **Server-Side Authentication**: Use Clerk's `currentUser()` for protected routes, never trust client tokens
- **Security Middleware**: Implement bot protection, rate limiting, and attack prevention with Arcjet
- **SSL by Default**: Automatic HTTPS configuration for production deployments
- **Structured Logging**: Log security events via LogTape but never expose internal errors to clients
- **Route Protection**: Use Next.js middleware for authentication guards and locale-aware security

```typescript
// Security middleware pipeline
export async function middleware(request: NextRequest) {
  // 1. Security layer (Arcjet)
  const decision = await aj.protect(request);
  
  // 2. Authentication layer (Clerk)
  const authResult = await clerkMiddleware(request);
  
  // 3. Internationalization layer
  return createI18nMiddleware(request);
}
```

### Error Handling
- **Structured Error Responses**: Use `z.treeifyError()` for consistent API error formatting
- **Client/Server Boundaries**: Handle errors differently in server components vs client components
- **Logging Integration**: Use LogTape for structured error logging with context
- **Fail Securely**: Never expose internal errors - sanitize all client-facing error messages
- **Form Validation**: Integrate React Hook Form errors with Zod validation schemas

```typescript
// API error handling pattern
if (!parse.success) {
  return NextResponse.json(z.treeifyError(parse.error), { status: 422 });
}

// Component error boundary pattern
if (form.formState.errors.increment) {
  return <div className="text-red-500">{t('error_increment_range')}</div>;
}
```

### Observable Systems & Logging Standards
- **Structured Logging**: Use LogTape with JSON format for machine-readable logs
- **Request Correlation**: Track user sessions with Clerk user IDs and request context
- **Multi-Output Logging**: Console for development, file/external services for production
- **Error Tracking**: Automatic error capture with Sentry integration via instrumentation
- **Analytics Integration**: PostHog for user behavior tracking with privacy-first configuration
- **Performance Monitoring**: Next.js built-in performance metrics and Sentry performance tracking

```typescript
// Logging with structured context
logger.info('Database operation completed', {
  operation: 'counter_update',
  userId: user?.id,
  timestamp: new Date().toISOString(),
  metadata: { increment: validatedData.increment }
});
```

### State Management
- **Server/Client Boundaries**: Use server components for data fetching, client components for interactivity
- **Form State**: React Hook Form for client-side form state with Zod validation
- **Database State**: Drizzle ORM with automatic TypeScript inference and hot-reload protection
- **Authentication State**: Clerk providers at layout boundaries with locale-aware configuration
- **Global Configuration**: Centralized AppConfig with environment-based service activation
- **Session Management**: Next.js session handling with Clerk integration

```typescript
// State composition pattern
const form = useForm({
  resolver: zodResolver(CounterValidation),
  defaultValues: { increment: 0 }
});

// Server state pattern
const result = await db.query.counterSchema.findMany({
  where: eq(counterSchema.id, id)
});
```

### API Design Principles (Next.js App Router)
- **Route Handlers**: Use Next.js 15 App Router API routes with proper TypeScript typing
- **HTTP Methods**: Export named functions (GET, POST, PUT, DELETE) from route.ts files
- **Input Validation**: Zod schema validation for all incoming request data
- **Error Responses**: Consistent error formatting with `z.treeifyError()` and proper status codes
- **Internationalization**: API routes support locale-aware responses via headers
- **Database Integration**: Direct database operations with Drizzle ORM type safety
- **Testing Isolation**: E2E testing patterns with `x-e2e-random-id` headers

```typescript
// Next.js API route pattern
export const PUT = async (request: Request) => {
  const json = await request.json();
  const parse = CounterValidation.safeParse(json);
  
  if (!parse.success) {
    return NextResponse.json(z.treeifyError(parse.error), { status: 422 });
  }
  
  return NextResponse.json({ count: result[0]?.count });
};
```


## 4. Multi-Agent Workflows & Context Injection

### Automatic Context Injection for Sub-Agents
When using the Task tool to spawn sub-agents, the core project context (CLAUDE.md, project-structure.md, docs-overview.md) is automatically injected into their prompts via the subagent-context-injector hook. This ensures all sub-agents have immediate access to essential project documentation without the need of manual specification in each Task prompt.


## 5. MCP Server Integrations

### Gemini Consultation Server
**When to use:**
- Complex coding problems requiring deep analysis or multiple approaches
- Code reviews and architecture discussions
- Debugging complex issues across multiple files
- Performance optimization and refactoring guidance
- Detailed explanations of complex implementations
- Highly security relevant tasks

**Automatic Context Injection:**
- The kit's `gemini-context-injector.sh` hook automatically includes two key files for new sessions:
  - `/docs/ai-context/project-structure.md` - Complete project structure and tech stack
  - `/MCP-ASSISTANT-RULES.md` - Your project-specific coding standards and guidelines
- This ensures Gemini always has comprehensive understanding of your technology stack, architecture, and project standards

**Usage patterns:**
```python
# New consultation session (project structure auto-attached by hooks)
mcp__gemini__consult_gemini(
    specific_question="How should I optimize this voice pipeline?",
    problem_description="Need to reduce latency in real-time audio processing",
    code_context="Current pipeline processes audio sequentially...",
    attached_files=[
        "src/core/pipelines/voice_pipeline.py"  # Your specific files
    ],
    preferred_approach="optimize"
)

# Follow-up in existing session
mcp__gemini__consult_gemini(
    specific_question="What about memory usage?",
    session_id="session_123",
    additional_context="Implemented your suggestions, now seeing high memory usage"
)
```

**Key capabilities:**
- Persistent conversation sessions with context retention
- File attachment and caching for multi-file analysis
- Specialized assistance modes (solution, review, debug, optimize, explain)
- Session management for complex, multi-step problems

**Important:** Treat Gemini's responses as advisory feedback. Evaluate the suggestions critically, incorporate valuable insights into your solution, then proceed with your implementation.

### Context7 Documentation Server
**Repository**: [Context7 MCP Server](https://github.com/upstash/context7)

**When to use:**
- Working with external libraries/frameworks (React, FastAPI, Next.js, etc.)
- Need current documentation beyond training cutoff
- Implementing new integrations or features with third-party tools
- Troubleshooting library-specific issues

**Usage patterns:**
```python
# Resolve library name to Context7 ID
mcp__context7__resolve_library_id(libraryName="react")

# Fetch focused documentation
mcp__context7__get_library_docs(
    context7CompatibleLibraryID="/facebook/react",
    topic="hooks",
    tokens=8000
)
```

**Key capabilities:**
- Up-to-date library documentation access
- Topic-focused documentation retrieval
- Support for specific library versions
- Integration with current development practices



## 6. Post-Task Completion Protocol
After completing any coding task, follow this checklist:

### 1. Type Safety & Quality Checks
Run the appropriate commands based on what was modified:
- **Python projects**: Run mypy type checking
- **TypeScript projects**: Run tsc --noEmit
- **Other languages**: Run appropriate linting/type checking tools

### 2. Verification
- Ensure all type checks pass before considering the task complete
- If type errors are found, fix them before marking the task as done