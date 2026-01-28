# Skills Hub Examples

## Basic Skill Invocation

### Explicit Skill Names

```bash
# Explain a file
/skills-hub explain src/api/auth.ts

# Suggest improvements
/skills-hub suggest src/utils/format.ts

# Brainstorm a decision
/skills-hub brainstorm should we use REST or GraphQL

# Update dependencies
/skills-hub update src/hooks/useAuth.ts

# Verify current information
/skills-hub grounded what are the React 19 breaking changes

# Generate instructions
/skills-hub write-for-claude create a CLAUDE.md for this project
```

### Intent-Based Routing

The hub infers the skill from natural language:

```bash
# Routed to explain
/skills-hub what does the authentication middleware do
/skills-hub how does error handling work in this codebase
/skills-hub why is getUserById caching for 5 minutes

# Routed to suggest
/skills-hub how can I improve src/api/users.ts
/skills-hub review the database layer for issues
/skills-hub find bugs in the payment processing code

# Routed to brainstorm
/skills-hub should we use a monorepo or separate repos
/skills-hub what are the trade-offs of JWT vs sessions
/skills-hub help me decide on a state management approach

# Routed to update
/skills-hub modernize the React Query hooks to latest version
/skills-hub upgrade the authentication to use the new Supabase patterns

# Routed to grounded
/skills-hub what's the current Next.js App Router best practice
/skills-hub are there any recent security advisories for lodash

# Routed to write-for-claude
/skills-hub write a skill spec for automated code review
/skills-hub create project instructions for the API team
```

## Skill Chaining

### explain → suggest (Understand, then improve)

```bash
/skills-hub explain src/api/users.ts then suggest improvements

# Output:
## Workflow: explain → suggest

### explain: src/api/users.ts

**What it does:** REST API handlers for user CRUD operations

**Key points:**
- Uses Express router with middleware chain
- Validates input with Zod schemas
- Returns standardized JSON responses

**Why it matters:** Central to user management, called by auth flows and admin dashboard

---

### suggest: Based on explanation

## Suggestions for src/api/users.ts

### 1. [HIGH] Add rate limiting to prevent abuse
**Category:** security
**Why:** Public endpoints without rate limiting are vulnerable to brute force

**Before:**
router.post('/users', validateUser, createUser);

**After:**
router.post('/users', rateLimit({ max: 10, windowMs: 60000 }), validateUser, createUser);

### 2. [MED] Extract validation middleware
**Category:** readability
...

---

### Summary

The users API is well-structured but needs rate limiting on write endpoints
and could benefit from extracting repeated validation logic into shared middleware.
```

### brainstorm → suggest (Decide, then implement)

> **Note:** brainstorm executes directly (not via Task) because it spawns its own Task agents.
> The hub runs brainstorm in the main session, then delegates suggest via Task with brainstorm's output.

```bash
/skills-hub brainstorm caching strategy for product catalog then suggest implementation

# Output:
## Workflow: brainstorm → suggest

### brainstorm: Caching Strategy

**Consensus Points:**
- Use Redis for distributed cache (team already has Redis)
- Cache at API response level, not database query level
- 5-minute TTL balances freshness with performance

**Key Trade-offs:**
| Perspective | Position | Accepts |
|-------------|----------|---------|
| Pragmatist | Redis with simple key structure | Manual invalidation on updates |
| Perfectionist | Redis with tag-based invalidation | Additional complexity |
| User Advocate | Aggressive caching with stale-while-revalidate | Occasional stale data |

**Recommendation:**
Redis caching with 5-minute TTL and explicit invalidation on product updates.
Use stale-while-revalidate pattern for non-critical reads.

---

### suggest: Implementation

## Suggestions for implementing caching

### 1. [HIGH] Add Redis cache wrapper
**Category:** performance

**Before:**
async function getProducts() {
  return db.products.findMany();
}

**After:**
async function getProducts() {
  const cached = await redis.get('products:all');
  if (cached) return JSON.parse(cached);

  const products = await db.products.findMany();
  await redis.setex('products:all', 300, JSON.stringify(products));
  return products;
}

### 2. [MED] Add cache invalidation on writes
...

---

### Summary

Brainstorm concluded Redis with 5-min TTL is the right approach.
Implementation requires cache wrapper, invalidation hooks, and monitoring.
```

### grounded → update (Verify, then modernize)

```bash
/skills-hub grounded check React Query v5 migration requirements then update src/hooks/useData.ts

# Output:
## Workflow: grounded → update

### grounded: React Query v5 Migration

**Verified (volatile technology info):**
- Package renamed: react-query → @tanstack/react-query
- useQuery signature changed: (key, fn, options) → ({ queryKey, queryFn, ...options })
- New features: useQueries, useSuspenseQuery
- Removed: isLoading replaced with isPending

**Trusted training (stable patterns):**
- Core React hooks concepts
- Caching fundamentals
- TypeScript patterns

**Key finding:** Migration requires updating all useQuery calls to object syntax.

---

### update: src/hooks/useData.ts

## Update Opportunities

### @tanstack/react-query v5 Migration
⚠️ BREAKING: Package renamed, API changes

📊 Summary:
- 🟢 2 low-risk changes (import updates)
- 🟡 4 medium-risk changes (API signatures)

### Changes:

1. 🟢 Import path update
- import { useQuery } from 'react-query'
+ import { useQuery } from '@tanstack/react-query'

2. 🟡 useQuery signature
- const { data, isLoading } = useQuery('users', fetchUsers);
+ const { data, isPending } = useQuery({ queryKey: ['users'], queryFn: fetchUsers });

...

---

### Summary

Verified current React Query v5 patterns via documentation.
Found 6 changes needed in useData.ts to complete migration.
```

## Parallel Execution

### Independent Targets

```bash
/skills-hub explain src/auth.ts and suggest improvements for src/utils.ts

# Both skills run in parallel since they target different files
# Output combines both results
```

### Multiple Explanations

```bash
/skills-hub explain the auth flow and the payment flow

# Parallel explain tasks for both topics
```

## Passing Skill Flags

All flags pass through to the target skill:

```bash
# explain with --deep
/skills-hub explain authentication flow --deep

# suggest with category filter
/skills-hub suggest --category security --depth thorough src/api/

# brainstorm with custom agents
/skills-hub brainstorm --agents security,performance,cost authentication approach

# update with dry-run
/skills-hub update --dry-run --max-risk low src/hooks/
```

## Complex Workflows

### Full Code Review

```bash
# Step 1: Understand
/skills-hub explain src/api/payments.ts --deep

# Step 2: Evaluate
/skills-hub suggest --depth thorough

# Step 3: Decide on approach
/skills-hub brainstorm how to address the security suggestions

# Step 4: Document decisions
/skills-hub write-for-claude document the security improvements plan
```

### Modernization Pipeline

```bash
# Step 1: Check current best practices
/skills-hub grounded what are the latest Express.js security best practices

# Step 2: Brainstorm approach
/skills-hub brainstorm how to modernize our Express middleware

# Step 3: Execute updates
/skills-hub update src/middleware/ --lib express
```

### Learning a Codebase

```bash
# Start high-level
/skills-hub explain the project structure --deep

# Dive into specific areas
/skills-hub explain the database layer
/skills-hub explain the authentication flow
/skills-hub explain the API routing

# Identify improvements
/skills-hub suggest --depth thorough src/
```

## Error Handling Examples

### Unknown Skill

```bash
/skills-hub debug src/api.ts

# Output:
Unknown skill: "debug"

Available skills:
- explain — understand what code does
- suggest — get improvement recommendations
- brainstorm — explore options from multiple perspectives
- update — modernize with current documentation
- grounded — verify time-sensitive information
- write-for-claude — generate Claude-optimized instructions

Did you mean: explain (to understand what's happening)?
```

### Ambiguous Intent

```bash
/skills-hub help me with src/api.ts

# Output:
I'm not sure which skill would help most here. What would you like to do?

- **explain** — Understand what the code does
- **suggest** — Get improvement recommendations
- **brainstorm** — Explore design options

Which approach fits your goal?
```

### Skill Failure in Chain

```bash
/skills-hub explain nonexistent-file.ts then suggest

# Output:
## Workflow: explain → suggest

### explain: nonexistent-file.ts

⚠️ Could not find nonexistent-file.ts

Similar files found:
- src/api/file.ts
- src/utils/files.ts

Options:
1. Retry with one of these files
2. Skip to suggest on a different target
3. Cancel workflow

Which would you like?
```

## Tips for Effective Use

### When to Chain

✅ Good chains:
- explain → suggest (understand before improving)
- brainstorm → write-for-claude (decide then document)
- grounded → update (verify before modernizing)

❌ Avoid:
- suggest → explain (improving before understanding)
- Long chains (>3 skills) — break into separate requests

**Note:** Chains involving brainstorm have special execution — brainstorm runs directly in the main session (not delegated) because it spawns its own Task agents internally.

### When to Run Parallel

✅ Independent targets:
```bash
/skills-hub explain src/auth.ts and explain src/payments.ts
```

❌ Dependent analysis:
```bash
# Don't parallelize — suggest needs explain context
/skills-hub explain src/api.ts then suggest
```

### Skill Selection Guide

| Your Goal | Use Skill |
|-----------|-----------|
| "What does this code do?" | explain |
| "What's wrong with this code?" | suggest |
| "Should we use X or Y?" | brainstorm |
| "Is this API still current?" | grounded |
| "Update to latest version" | update |
| "Document this for the team" | write-for-claude |
