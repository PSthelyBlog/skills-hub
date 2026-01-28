# Real-World Examples

Practical examples of using the `/explain` skill in common development scenarios.

---

## Scenario 1: Joining a New Codebase

**Day 1: Getting oriented**

```
You: /explain project structure

Claude:
## Project Structure

**What it does**: Organizes code into src/ (application), tests/ (specs), and config/ (settings)

**Key points**:
- src/controllers handle HTTP requests
- src/services contain business logic
- src/models define database schemas

**Why it matters**: Controllers call services, services call models. Data flows down, never up.

---
Need more? Add --deep for architectural patterns and conventions.
```

**Week 1: Understanding a critical module**

```
You: /explain authentication flow --deep

Claude:
## Authentication Flow

**Overview**: Multi-layered auth using JWT tokens with Redis session storage
and role-based access control. Tokens expire after 24h, refresh tokens last 30 days.

**How it works**:
[... comprehensive 7-section breakdown ...]
```

---

## Scenario 2: Debugging a Production Issue

**Step 1: Understand the error**

```
You: /explain TypeError: Cannot read property 'id' of undefined at UserController.getProfile

Claude:
## TypeError in UserController.getProfile

**What it does**: This error means req.user is undefined when the code tries to access req.user.id

**Key points**:
- req.user is set by auth middleware
- If undefined, middleware didn't run or token was invalid
- Happens on lines accessing user data without checking

**Why it matters**: Protected routes need authentication middleware in the route definition.

---
Need more? Add --deep for middleware chain analysis and debugging steps.
```

**Step 2: Investigate deeply**

```
You: /explain why is req.user undefined --deep

Claude:
## Why req.user is undefined

**Overview**: The auth middleware isn't executing before UserController.getProfile,
likely due to missing middleware in route configuration or incorrect route ordering.

**How it works**:
1. Express routes execute middleware in order
2. authMiddleware checks JWT and sets req.user
3. If authMiddleware skipped, req.user never gets set
4. Controller assumes req.user exists

**Dependencies**:
- Routes defined in: routes/user.js
- Middleware: middleware/auth.js
- Protected routes need: router.get('/profile', authMiddleware, controller.getProfile)

[... continues with debugging steps ...]
```

---

## Scenario 3: Refactoring Legacy Code

**Before making changes**

```
You: /explain what does syncUsers do

Claude:
## syncUsers

**What it does**: Pulls user data from external CRM and updates local database

**Key points**:
- Runs on cron schedule (daily at 2am)
- Updates existing users, creates new ones
- Logs sync failures to monitoring service

**Why it matters**: Don't modify during business hours - affects customer data.

---
Need more? Add --deep for data flow and failure modes.
```

**Understanding impact**

```
You: /explain what calls syncUsers

Claude:
## syncUsers Callers

**What it does**: syncUsers is called by 2 sources

**Key points**:
- jobs/nightly-sync.js - scheduled daily sync
- api/admin/force-sync endpoint - manual trigger for admins
- Also indirectly triggered by webhooks from CRM

**Why it matters**: Changes affect scheduled jobs AND admin panel functionality.

---
Need more? Add --deep for full dependency graph and side effects.
```

```
You: /explain syncUsers side effects --deep

Claude:
## syncUsers Side Effects

**Overview**: syncUsers modifies 3 database tables, triggers email notifications,
updates cache, and logs to external monitoring. Changes cascade through user-facing features.

**How it works**:
[... detailed analysis of every side effect ...]

**Gotchas**:
- Disables cache temporarily (can cause slow queries during sync)
- Email notifications queue can overflow on large syncs
- Foreign key constraints can cause sync failures if data order is wrong

[... continues with safe refactoring approach ...]
```

---

## Scenario 4: Code Review

**Reviewing a PR**

```
You: /explain what changed in processPayment between main and feature-branch --deep

Claude:
## Changes to processPayment

**Overview**: PR adds retry logic and idempotency key support to prevent duplicate charges.
Three key changes: retry on network errors, idempotency keys for safety, better error logging.

**How it works**:
Before: Direct Stripe call, throws on any error
After: Retry logic, idempotency keys, null returns instead of throws

**Design choices**:
- Single retry (balance reliability vs latency)
- Idempotency keys prevent duplicate charges if retry hits already-successful call
- Null return vs throwing prevents cascading failures in checkout flow

**Gotchas**:
- Callers now need to check for null returns (breaking change if they expect throws)
- Idempotency keys only work if order ID is stable
- Retry adds 2s latency on first failure

**Impact analysis**:
- checkout-controller needs update (expects throws, now gets null)
- Tests need new cases (retry scenarios, idempotency)
- Monitoring should track retry rate

**Recommendation**: Request error handling updates in calling code before merge.
```

---

## Scenario 5: Performance Investigation

**Initial question**

```
You: /explain why is the dashboard slow

Claude:
## Dashboard Performance

**What it does**: The dashboard queries 5 database tables and processes results in memory

**Key points**:
- No query caching (hits DB on every load)
- Joins 5 tables (users, orders, products, reviews, analytics)
- Processes 10k+ records in JavaScript after fetch

**Why it matters**: Query takes 2-3 seconds, processing takes another 1-2 seconds.

---
Need more? Add --deep for query analysis and optimization opportunities.
```

**Deep investigation**

```
You: /explain dashboard performance --deep

Claude:
## Dashboard Performance Analysis

**Overview**: Dashboard is slow due to unoptimized queries (missing indexes),
N+1 query pattern in related data, and client-side aggregation of data that
should be handled by database.

**How it works**:
[... detailed query breakdown ...]

**Performance characteristics**:
- Main query: 1.8s (no index on date filter)
- N+1 queries: 0.5s (fetches related data in loop)
- Client aggregation: 0.9s (JavaScript processing)
- Total: ~3.2s average

**Optimization opportunities**:
1. Add composite index on (user_id, created_at) - saves ~1.5s
2. Use JOIN instead of N+1 queries - saves ~0.5s
3. Move aggregation to SQL (SUM, GROUP BY) - saves ~0.8s
4. Add Redis caching (5min TTL) - saves ~2.8s after first load

**Trade-offs**:
- Index: Write speed impact minimal, read speed gains massive
- JOIN: More complex query, but single DB round trip
- SQL aggregation: Standard practice, more efficient
- Caching: Stale data risk (acceptable for dashboards)

**Implementation priority**:
1. Add index (easy, safe, immediate impact)
2. Fix N+1 (medium effort, big impact)
3. Move aggregation (refactor, test thoroughly)
4. Add caching (last - masks underlying issues)

**Related**:
- Similar pattern in reports module (can apply same fixes)
- database/indexes.sql for index definitions
```

---

## Scenario 6: Learning a Pattern

**Understanding middleware pattern**

```
You: /explain how middleware works in this codebase --deep

Claude:
## Middleware Pattern

**Overview**: Express middleware functions execute in sequence before route handlers,
providing authentication, logging, validation, and error handling as reusable layers.

**How it works**:
1. Request arrives at Express
2. Middleware chain executes in order (app.use, router.use)
3. Each middleware can: modify req/res, end request, or call next()
4. If all middleware calls next(), route handler executes
5. Error middleware catches thrown errors or next(err) calls

**Common patterns in this codebase**:

**Authentication middleware** (middleware/auth.js):
```javascript
// Validates JWT, sets req.user, or returns 401
function authMiddleware(req, res, next) {
  const token = req.headers.authorization;
  if (!token) return res.status(401).json({error: 'No token'});

  try {
    req.user = jwt.verify(token, SECRET);
    next(); // Continue to route handler
  } catch (err) {
    res.status(401).json({error: 'Invalid token'});
  }
}
```

**Validation middleware** (middleware/validate.js):
```javascript
// Checks request body against schema
function validate(schema) {
  return (req, res, next) => {
    const {error} = schema.validate(req.body);
    if (error) return res.status(400).json({error: error.details});
    next(); // Valid, continue
  };
}
```

**Usage in routes**:
```javascript
router.post('/users',
  authMiddleware,           // First: check auth
  validate(userSchema),     // Second: validate data
  userController.create     // Finally: create user
);
```

**Design choices**:
- Middleware chain = separation of concerns (auth separate from business logic)
- next() pattern = explicit control flow
- Error middleware at end = centralized error handling
- Order matters = auth before validation before handler

**Gotchas**:
- Forgetting next() = request hangs forever
- Order matters = validate before auth won't work (no req.user)
- Error middleware needs 4 params = (err, req, res, next)
- Async middleware needs error handling = wrap in try-catch or use express-async-errors

**Related patterns**:
- Similar to Python WSGI middleware
- Express.js convention used throughout
- See middleware/ directory for all available middleware
```

---

## Scenario 7: Quick Lookups During Development

**Rapid-fire questions**

```
You: /explain formatDate

Claude:
## formatDate

**What it does**: Converts Date objects to YYYY-MM-DD strings

**Key points**:
- Returns null for invalid dates
- Uses UTC timezone (not local)
- Pads single-digit months/days with zeros

**Why it matters**: Safe for database inserts (PostgreSQL date format).
```

```
You: /explain isAdmin

Claude:
## isAdmin

**What it does**: Checks if user has admin role

**Key points**:
- Returns true if user.role === 'admin' OR user.role === 'superadmin'
- Returns false for null/undefined users (safe to call)
- Case-sensitive role check

**Why it matters**: Use this instead of direct role checks for consistency.
```

```
You: /explain API_BASE_URL

Claude:
## API_BASE_URL

**What it does**: Environment-specific API endpoint constant

**Key points**:
- Development: http://localhost:3000
- Staging: https://api-staging.example.com
- Production: https://api.example.com

**Why it matters**: Don't hardcode endpoints - use this constant for all API calls.
```

---

## Pattern Summary

### Quick Mode Usage Patterns
- ✅ "What does X do" questions
- ✅ Single function/variable lookups
- ✅ Quick error message context
- ✅ Rapid exploration during active coding

### Deep Mode Usage Patterns
- ✅ Understanding architecture before refactoring
- ✅ Debugging complex issues
- ✅ Code review impact analysis
- ✅ Learning unfamiliar subsystems
- ✅ Performance investigation

### Progressive Workflow
1. Start with quick explain
2. Scan the key points
3. If you need more, add --deep
4. Get comprehensive analysis

### Chaining Pattern
```
/explain [topic]           # Understand
[review explanation]       # Learn
[make changes]             # Apply
[test]                     # Verify
/explain [related topic]   # Expand knowledge
```

---

**The best explanation is the one that helps you move forward with confidence.**

Use quick mode to move fast. Use deep mode to move safely.
