# Real-World Examples

Practical examples of using the `/brainstorm` skill in common development scenarios.

**Remember**: Invoke `/brainstorm` directly, not via skills-hub delegation.

---

## Scenario 1: Choosing an API Style

**The decision**

```
You: /brainstorm REST vs GraphQL for our public API

Claude:
[Spawns 3 parallel Task agents - Round 1]

🎭 The Pragmatist is drafting a proposal...
🎭 The Perfectionist is analyzing trade-offs...
🎭 The User Advocate is considering impact...

[Round 1 completes]

[Spawns 3 parallel Task agents - Round 2]

🎭 Agents are reviewing each other's positions...

[Round 2 completes]

## Brainstorm: REST vs GraphQL for Public API

**Consensus Points**
- Start with REST for v1 (simpler to ship and document)
- Design resources to be composable (enables GraphQL later if needed)
- Version the API from day one (/v1/ prefix)

**Key Trade-offs**
| Perspective | Position | Accepts |
|-------------|----------|---------|
| Pragmatist | REST | More endpoints, but faster to ship |
| Perfectionist | REST with OpenAPI | Documentation overhead, but better DX |
| User Advocate | REST | Users know REST; GraphQL learning curve |

**Unresolved Tensions**
- Mobile clients may need multiple requests (GraphQL advantage)
- If data relationships are complex, REST can get unwieldy

**Recommendation**
Start with REST + OpenAPI spec. Design resources to be composable
(e.g., /users/{id}?include=orders,profile). This gives you REST simplicity
now with a migration path to GraphQL if client needs evolve.

**Suggested Next Steps**
1. Define core resources and relationships in OpenAPI
2. Implement ?include= pattern for nested data
3. Monitor client usage - if N+1 request patterns emerge, revisit GraphQL
```

---

## Scenario 2: State Management Architecture

**Getting multiple perspectives on a complex choice**

```
You: /brainstorm --agents pragmatist,perfectionist,innovator state management for React app

Claude:
[Spawns 3 parallel Task agents with custom personas]

🎭 The Pragmatist is drafting a proposal...
🎭 The Perfectionist is analyzing trade-offs...
🎭 The Innovator is exploring alternatives...

[Rounds complete]

## Brainstorm: State Management for React App

**Consensus Points**
- Start with React Context + useReducer (built-in, no dependencies)
- Extract to dedicated solution only when you hit pain points
- Server state (API data) should use React Query or similar

**Key Trade-offs**
| Perspective | Position | Accepts |
|-------------|----------|---------|
| Pragmatist | Context + useReducer | Prop drilling for deep trees |
| Perfectionist | Zustand | Another dependency, but cleaner API |
| Innovator | Server-first with React Query | Less client state, paradigm shift |

**Unresolved Tensions**
- Context re-renders can hurt performance in large apps
- Team familiarity varies (some know Redux, others don't)

**Recommendation**
Split state into two categories:
1. **Server state**: Use React Query (caching, invalidation handled)
2. **UI state**: Start with Context, migrate to Zustand if needed

This avoids premature complexity while giving you escape hatches.

**Suggested Next Steps**
1. Install React Query for API data
2. Create a single UI context for global UI state (theme, sidebar, etc.)
3. Keep component-local state local (useState)
4. Revisit in 3 months - if Context causes pain, evaluate Zustand
```

---

## Scenario 3: Security-Focused Authentication

**Custom agents for domain-specific perspectives**

```
You: /brainstorm --agents security,pragmatist,user-advocate authentication for mobile app

Claude:
[Spawns 3 parallel Task agents]

🎭 The Security Expert is drafting a proposal...
🎭 The Pragmatist is drafting a proposal...
🎭 The User Advocate is drafting a proposal...

[Rounds complete]

## Brainstorm: Authentication for Mobile App

**Consensus Points**
- Use OAuth 2.0 + PKCE (industry standard for mobile)
- Store tokens in secure storage (Keychain/Keystore), not AsyncStorage
- Implement biometric unlock for convenience without compromising security

**Key Trade-offs**
| Perspective | Position | Accepts |
|-------------|----------|---------|
| Security | OAuth 2.0 + PKCE + short tokens | More refreshes, but limits exposure |
| Pragmatist | OAuth 2.0 + PKCE | Implementation complexity for security |
| User Advocate | Biometric + session persistence | Security trade-off for UX |

**Unresolved Tensions**
- Biometric unlock reduces security slightly (User Advocate priority)
- Short token expiry (15min) may cause UX friction (Security priority)

**Recommendation**
OAuth 2.0 with PKCE flow:
- Access tokens: 15 minutes
- Refresh tokens: 30 days, rotated on use
- Biometric unlock: Optional, re-authenticates with stored refresh token
- Secure storage: Keychain (iOS), Keystore (Android)

**Suggested Next Steps**
1. Choose OAuth provider (Auth0, Cognito, or custom)
2. Implement PKCE flow with secure token storage
3. Add biometric unlock as opt-in feature
4. Set up token refresh middleware to handle expiry transparently
```

---

## Scenario 4: Monorepo vs Polyrepo

**Classic architecture decision**

```
You: /brainstorm monorepo vs separate repos for our 3-person startup

Claude:
[Spawns 3 parallel Task agents]

## Brainstorm: Monorepo vs Separate Repos

**Consensus Points**
- Monorepo wins for a 3-person team (unanimous)
- Establish module boundaries early (prevents spaghetti)
- Use simple tooling (npm workspaces) over complex (Nx, Bazel)

**Key Trade-offs**
| Perspective | Position | Accepts |
|-------------|----------|---------|
| Pragmatist | Monorepo + npm workspaces | CI runs everything on every PR |
| Perfectionist | Monorepo + turborepo | More tooling, but smarter CI |
| User Advocate | Monorepo | Onboarding is one git clone |

**Unresolved Tensions**
- Will need to optimize CI if repo grows large (>50 packages)
- Some external tools assume separate repos

**Recommendation**
Monorepo with npm workspaces:
- Structure: apps/ (deployables) + packages/ (shared code)
- CI: Run all tests for now; add affected-only when it gets slow
- Boundaries: Clear package.json for each module

At 3 people, coordination cost of polyrepo outweighs any benefit.

**Suggested Next Steps**
1. Create repo structure: apps/, packages/, shared configs
2. Set up npm workspaces in root package.json
3. Add basic CI (test all packages on PR)
4. Document module boundaries in ARCHITECTURE.md
```

---

## Scenario 5: Refactoring a Large File

**Planning a code change**

```
You: /brainstorm how to break up this 2000-line UserService.ts

Claude:
## Brainstorm: Breaking Up 2000-line UserService.ts

**Consensus Points**
- Split by domain responsibility, not by method count
- Keep the main UserService as a facade (preserves API)
- Extract one piece at a time, not all at once

**Key Trade-offs**
| Perspective | Position | Accepts |
|-------------|----------|---------|
| Pragmatist | Facade + 3 extracted services | More files, but manageable |
| Perfectionist | Domain-driven split + interfaces | More upfront design, but cleaner |
| User Advocate | Incremental extraction | Slower, but less risky |

**Unresolved Tensions**
- How to handle cross-cutting concerns (logging, validation)?
- Tests are currently coupled to UserService - need refactoring too

**Recommendation**
Incremental extraction behind a facade:

1. **UserAuthService**: login, logout, password reset, 2FA
2. **UserProfileService**: CRUD, preferences, avatar
3. **UserNotificationService**: email preferences, push settings

Keep UserService as facade that delegates to these. Callers don't change.

**Suggested Next Steps**
1. Identify method clusters by scanning the file
2. Extract UserAuthService first (most isolated)
3. Update tests to use extracted service directly
4. Repeat for Profile, then Notifications
5. Decide later if facade should remain or be deprecated
```

---

## Scenario 6: Testing Strategy

**Process decision with team impact**

```
You: /brainstorm testing strategy for a new microservice

Claude:
## Brainstorm: Testing Strategy for New Microservice

**Consensus Points**
- Integration tests > unit tests for microservices (test real behavior)
- Use testcontainers for database/Redis dependencies
- Write fewer, more meaningful tests (not chasing coverage %)

**Key Trade-offs**
| Perspective | Position | Accepts |
|-------------|----------|---------|
| Pragmatist | Integration-heavy, few units | Slower tests, but higher confidence |
| Perfectionist | Testing pyramid + contract tests | More test types, but catches more |
| User Advocate | Focus on user journeys | Less edge coverage, but tests what matters |

**Unresolved Tensions**
- Integration tests are slower (CI time concern)
- Contract tests require coordination with other teams

**Recommendation**
Testing trophy (not pyramid) approach:

1. **Many integration tests**: Test API endpoints with real DB (testcontainers)
2. **Few unit tests**: Only for complex business logic
3. **E2E tests**: Critical user journeys only (3-5 tests)
4. **Contract tests**: If consuming/providing APIs to other services

Skip mocking databases - testcontainers gives you real behavior.

**Suggested Next Steps**
1. Set up testcontainers for PostgreSQL/Redis
2. Write integration tests for each API endpoint
3. Add unit tests only for pure business logic (calculations, validations)
4. Create E2E suite for critical paths (auth, main feature)
```

---

## Scenario 7: Performance Optimization

**Using the performance persona**

```
You: /brainstorm --agents performance,pragmatist,user-advocate optimizing slow dashboard

Claude:
## Brainstorm: Optimizing Slow Dashboard

**Consensus Points**
- Measure before optimizing (profile first)
- Start with backend (usually higher ROI than frontend)
- Consider perceived performance, not just actual speed

**Key Trade-offs**
| Perspective | Position | Accepts |
|-------------|----------|---------|
| Performance | Database indexes + query optimization | More DB resources |
| Pragmatist | Caching layer (Redis) | Stale data risk, but quick win |
| User Advocate | Skeleton loading + progressive render | Feels faster even if not |

**Unresolved Tensions**
- Caching introduces stale data (acceptable for dashboard?)
- Full optimization may require schema changes

**Recommendation**
Three-phase approach:

**Phase 1 (Quick wins)**:
- Add missing database indexes
- Implement skeleton loading (perceived performance)

**Phase 2 (Caching)**:
- Add Redis cache for expensive aggregations
- 5-minute TTL is acceptable for dashboard data

**Phase 3 (If needed)**:
- Profile specific slow queries
- Consider materialized views for complex aggregations

**Suggested Next Steps**
1. Add APM/profiling to identify slowest endpoints
2. Add composite index on (user_id, created_at) - likely missing
3. Implement skeleton loading in frontend
4. Add Redis caching for aggregation endpoints
```

---

## Pattern Summary

### When to Use Default Agents
- General design decisions
- Architecture choices
- Process/workflow questions

### When to Use Custom Agents
```bash
# Security-critical decisions
/brainstorm --agents security,pragmatist,user-advocate ...

# Performance-focused
/brainstorm --agents performance,pragmatist,skeptic ...

# User experience focus
/brainstorm --agents user-advocate,innovator,pragmatist ...

# Risk assessment
/brainstorm --agents skeptic,security,perfectionist ...
```

### Chaining Pattern
```bash
# Brainstorm the approach
/brainstorm how to implement feature X

# [choose an approach]

# Explain the details
/explain how [chosen approach] works --deep

# [implement with confidence]
```

---

## How to Verify True Multi-Agent Execution

**Correct behavior**: You see 3 separate Task results being collected, then synthesized.

**Incorrect behavior (theatrical fallback)**: You see a dialogue script where agents "talk" to each other in a narrative format.

If you see theatrical output, you're likely invoking via delegation instead of directly.

---

**The best brainstorm surfaces tensions you hadn't considered and gives you confidence in your decision.**
