# Examples: Before and After

Real-world examples showing human-style documentation transformed into Claude-optimized instructions.

---

## Example 1: Project CLAUDE.md

### Before (Human-Style)

```markdown
# Project Guidelines

This is a Node.js API project. We care about code quality and testing.

Please try to follow our existing patterns when adding new code.
Keep things simple and maintainable. Tests are important, especially
for business logic. Use TypeScript properly - we want type safety.

Error handling should be robust. Don't forget logging for debugging.
Performance matters but don't optimize prematurely.

When in doubt, look at how similar things are done elsewhere in the codebase.
```

### After (Claude-Optimized)

```markdown
# Project Guidelines

## Context

Node.js REST API serving B2B customers. TypeScript strict mode. Jest for testing.
Critical paths: authentication, billing, data export.

## Directives

### Code Style

MUST: Use TypeScript strict mode (`strict: true` in tsconfig)
MUST: Match formatting of adjacent code in same file
MUST: Use existing utilities from `src/utils/` before creating new ones
SHOULD: Keep functions under 40 lines
SHOULD: Keep files under 300 lines
SHOULD: Prefer named exports over default exports
MAY: Extract repeated logic (>5 lines, >2 occurrences) into utility

### Testing

MUST: Write unit tests for new functions in `src/services/`
MUST: Write integration tests for new API endpoints
MUST: Achieve >80% line coverage for new files
MUST: Run `npm test` before committing (all tests must pass)
SHOULD: Add edge case tests when fixing bugs
SHOULD: Mock external services, not internal modules
MAY: Add performance benchmarks for functions in hot paths

### Error Handling

MUST: Use typed errors from `src/errors/` for known error cases
MUST: Log errors with context: `{ error, requestId, userId, operation }`
MUST: Return appropriate HTTP status codes:
  - 400: Validation errors (include field-level messages)
  - 401: Authentication failures (no details in response)
  - 403: Authorization failures (no details in response)
  - 404: Resource not found
  - 500: Unexpected errors (log full stack, return requestId only)
MUST NOT: Expose stack traces in production responses
MUST NOT: Log sensitive data (passwords, tokens, PII)

### Performance

MUST: Profile with `clinic.js` before optimizing
MUST NOT: Optimize without measured performance data
SHOULD: Use database indexes for queries filtering >1000 rows
SHOULD: Batch database operations when processing >10 records
MAY: Add caching for data that changes <1x per hour

### Patterns to Follow

MUST: Use repository pattern for database access (`src/repositories/`)
MUST: Use service layer for business logic (`src/services/`)
MUST: Use controller layer for HTTP handling (`src/controllers/`)
MUST: Use dependency injection via constructor parameters
MUST NOT: Import repositories directly in controllers
MUST NOT: Put business logic in controllers

## Definitions

BUSINESS LOGIC: Operations involving validation rules, calculations, or data transformations beyond simple CRUD
HOT PATH: Code executed >100 times per second under normal load
UTILITY: Pure function with no side effects, reusable across modules
SIMILAR THINGS: Implementations in same directory with same purpose (e.g., other service methods)

## Anti-Patterns

MUST NOT: Use `any` type (use `unknown` and narrow with type guards)
MUST NOT: Catch errors without logging or re-throwing
MUST NOT: Create abstraction used by only one caller
MUST NOT: Add configuration for hypothetical future needs
MUST NOT: Use synchronous file operations in request handlers

## Decision Trees

### Adding New Functionality

IF similar functionality exists in codebase:
  THEN follow that implementation pattern exactly
ELSE IF established pattern exists in Node.js ecosystem:
  THEN follow ecosystem convention (cite source in PR)
ELSE:
  THEN propose approach in PR description for review

### Choosing Test Type

IF pure function with no dependencies:
  THEN unit test only
ELSE IF service with database dependency:
  THEN unit test with mocked repository + integration test
ELSE IF API endpoint:
  THEN integration test with test database
ELSE IF external API integration:
  THEN unit test with mocked client + contract test
```

### What Changed

| Human Version | Problem | Claude-Optimized Version |
|---------------|---------|--------------------------|
| "care about code quality" | Undefined term | Specific metrics: line limits, coverage % |
| "follow our existing patterns" | Which patterns? | Explicit: repository, service, controller layers |
| "keep things simple" | Subjective | Measurable: <40 line functions, <300 line files |
| "tests are important" | How important? Which tests? | MUST for specific scopes, coverage requirements |
| "use TypeScript properly" | Undefined | Strict mode, no `any`, typed errors |
| "robust error handling" | Vague | Enumerated status codes, logging requirements |
| "when in doubt" | Requires judgment | Decision tree with explicit conditions |

---

## Example 2: Agent Definition

### Before (Human-Style)

```markdown
# Code Review Agent

This agent reviews pull requests. It should look for bugs, style issues,
and potential improvements. Be thorough but not too nitpicky - focus on
things that actually matter. If you see security issues, definitely flag those.

Try to be helpful and constructive in feedback.
```

### After (Claude-Optimized)

```markdown
## Agent: Code Reviewer

### Role

Analyze pull request changes for correctness, security, and maintainability.

### Scope

MUST: Review only files modified in the PR
MUST: Focus on changed lines and their immediate context (10 lines above/below)
MUST NOT: Comment on unchanged code
MUST NOT: Review files outside the PR diff

### Responsibilities

MUST: Flag logical errors that would cause incorrect behavior
MUST: Flag security vulnerabilities (see Security Checklist)
MUST: Flag missing error handling for failure cases
MUST: Verify new functions have corresponding tests
SHOULD: Flag style violations if >3 occurrences in PR
SHOULD: Note missing edge case handling
SHOULD: Suggest clearer names for ambiguous identifiers
MAY: Suggest performance improvements for O(n^2)+ operations
MAY: Suggest refactoring if touched code has clear improvement opportunity

### Security Checklist

MUST flag:
- SQL injection: string concatenation in queries
- XSS: unescaped user input in HTML output
- Command injection: user input in shell commands
- Path traversal: user input in file paths
- Hardcoded secrets: API keys, passwords, tokens in code
- Missing authentication: endpoints without auth middleware
- Missing authorization: operations without permission checks

### Severity Classification

**BLOCKING (must fix before merge):**
- Logical errors that cause wrong output
- Security vulnerabilities
- Breaking changes without version bump

**NON-BLOCKING (should fix, PR can merge):**
- Missing test coverage
- Style violations
- Documentation gaps
- Performance issues in non-critical paths

**ADVISORY (nice to have):**
- Refactoring suggestions
- Alternative approaches
- Future improvement ideas

### Output Format

```markdown
## Code Review: [PR Title]

### Blocking Issues
[List each with: file:line | issue | suggested fix]
[If none: "No blocking issues found."]

### Non-Blocking Issues
[List each with: file:line | issue | suggested fix]
[If none: "No non-blocking issues found."]

### Advisory
[Optional suggestions for improvement]

### Summary
- Files reviewed: [count]
- Lines changed: [+added/-removed]
- Test coverage: [new functions tested: X/Y]
- Security checks: [passed/failed]
```

### Handoff Conditions

WHEN changes touch authentication/authorization code:
  Flag for Security Reviewer (in addition to standard review)

WHEN changes touch database schema:
  Flag for DBA review (in addition to standard review)

WHEN changes touch payment processing:
  Flag for Payment Team review (in addition to standard review)

WHEN PR is documentation-only (*.md files, comments only):
  Auto-approve with "Documentation changes only - no code review required"

### Feedback Tone

MUST: State issue objectively: "[What's wrong] because [why it matters]"
MUST: Include suggested fix for every issue raised
MUST NOT: Use subjective criticism ("this is bad", "I don't like")
MUST NOT: Use rhetorical questions ("why would you do this?")
SHOULD: Acknowledge good patterns: "Good use of [pattern] here"
```

### What Changed

| Human Version | Problem | Claude-Optimized Version |
|---------------|---------|--------------------------|
| "look for bugs" | Which bugs? | Enumerated: logical errors, security issues |
| "style issues" | Which styles matter? | Specific: flag if >3 occurrences |
| "not too nitpicky" | Subjective threshold | Severity levels with clear criteria |
| "things that actually matter" | Undefined | Blocking vs non-blocking classification |
| "definitely flag security" | Which security issues? | Explicit checklist of 7 categories |
| "be helpful and constructive" | Vague tone guidance | Specific format: issue + reason + fix |

---

## Example 3: Skill Specification

### Before (Human-Style)

```markdown
# Commit Message Skill

Helps write good commit messages. Should look at the changes and summarize
what was done. Follow conventional commit format if the project uses it.
Keep messages concise but informative.
```

### After (Claude-Optimized)

```markdown
# Commit Message Skill

---
name: commit-message
description: "Generate conventional commit messages from staged changes"
version: 1.0.0
model: haiku
---

## Purpose

Generate a commit message from staged git changes following conventional commit format.

## Protocol

### Step 1: Analyze Staged Changes

Execute: `git diff --cached --stat` and `git diff --cached`

Extract:
- Files modified (list)
- Total lines added/removed
- Nature of changes (what was done)

IF no staged changes:
  RETURN "No staged changes. Stage files with `git add` first."

### Step 2: Determine Commit Type

Based on changes, select ONE type:

| Type | Use When |
|------|----------|
| feat | New user-facing functionality |
| fix | Bug fix for user-facing behavior |
| refactor | Code change that neither fixes nor adds functionality |
| test | Adding or modifying tests only |
| docs | Documentation only changes |
| style | Formatting, whitespace (no logic change) |
| chore | Build, CI, dependencies, config |
| perf | Performance improvement |

IF changes span multiple types:
  Use type of the primary change (most lines changed)
  Note secondary changes in body

### Step 3: Identify Scope (Optional)

Scope = directory or module most affected

IF changes span single directory: scope = directory name
IF changes span single module: scope = module name
IF changes span multiple unrelated areas: omit scope

### Step 4: Write Subject Line

Format: `<type>(<scope>): <description>`

Rules:
- MUST: Use imperative mood ("add" not "added")
- MUST: Start with lowercase
- MUST: No period at end
- MUST: Keep under 50 characters
- MUST: Describe WHAT changed, not HOW

### Step 5: Write Body (If Needed)

IF changes are self-explanatory from diff: omit body
IF changes require context: add body

Body rules:
- MUST: Blank line between subject and body
- MUST: Wrap at 72 characters
- SHOULD: Explain WHY, not WHAT (diff shows what)
- SHOULD: Note breaking changes
- MAY: Reference issues: "Fixes #123"

### Step 6: Output Format

```
<type>(<scope>): <subject>

<body - optional>

<footer - optional>
```

## Examples

### Example 1: Simple Feature

**Staged changes:** New file `src/utils/formatDate.ts` (25 lines)

**Output:**
```
feat(utils): add date formatting utility

Formats dates to ISO 8601 for API responses.
```

### Example 2: Bug Fix

**Staged changes:** Modified `src/auth/login.ts` (3 lines changed)

**Output:**
```
fix(auth): handle expired tokens in login flow
```

### Example 3: Multiple Files

**Staged changes:**
- `src/api/users.ts` (+50 lines)
- `src/api/users.test.ts` (+100 lines)
- `docs/api.md` (+20 lines)

**Output:**
```
feat(api): add user search endpoint

Implements GET /users/search with filtering by name and email.
Includes pagination support.

Closes #456
```

### Example 4: Refactor

**Staged changes:** Multiple files in `src/services/` renamed and reorganized

**Output:**
```
refactor(services): reorganize service layer structure

Split monolithic UserService into focused modules:
- UserAuthService: authentication logic
- UserProfileService: profile management
- UserPreferencesService: settings handling

No behavior changes.
```

## Edge Cases

**Mixed changes (feat + fix):**
  Use primary type, note secondary in body:
  ```
  feat(api): add rate limiting

  Also fixes memory leak in request counting (#789).
  ```

**No clear scope:**
  Omit scope:
  ```
  chore: update dependencies to latest versions
  ```

**Breaking change:**
  Add footer:
  ```
  feat(api)!: change authentication to JWT

  BREAKING CHANGE: Token format changed from session cookie to JWT.
  Clients must update authentication headers.
  ```

**Empty diff but staged file:**
  RETURN "File staged but no changes detected. Check with `git diff --cached`."

## Validation Checklist

Before outputting, verify:
- [ ] Type is one of the allowed types
- [ ] Subject is imperative mood
- [ ] Subject under 50 characters
- [ ] Subject has no period
- [ ] Body wrapped at 72 characters (if present)
- [ ] Breaking changes noted in footer (if applicable)
```

### What Changed

| Human Version | Problem | Claude-Optimized Version |
|---------------|---------|--------------------------|
| "look at changes" | How? | Specific git commands to run |
| "summarize what was done" | What detail level? | Exact format: type(scope): subject |
| "if project uses it" | How to detect? | Always use conventional commits |
| "concise but informative" | How concise? | 50 char subject, 72 char body |
| (missing) | No edge cases | Explicit handling for mixed, breaking, empty |
| (missing) | No examples | Four example inputs/outputs |

---

## Example 4: Simple Instructions

### Before (Human-Style)

```markdown
When you see TODOs in the code, you should probably address them if they're
related to what you're working on. Use your judgment about whether they're
important enough to fix now.
```

### After (Claude-Optimized)

```markdown
## TODO Handling

### When to Address TODOs

MUST address if:
- TODO is in a function you are modifying
- TODO describes the exact task you were asked to do
- TODO blocks completion of current task

SHOULD address if:
- TODO is in a file you are modifying
- TODO is trivial (<5 minute fix)
- TODO is related to current task scope

MAY defer if:
- TODO is unrelated to current task
- TODO requires architectural decision
- TODO has "FUTURE:" prefix

MUST NOT address if:
- TODO references external dependency or team decision
- TODO has issue number reference (e.g., "TODO(#123)")

### When Addressing TODOs

MUST: Remove the TODO comment after implementing
MUST: Test that the TODO implementation works
SHOULD: Note in commit message: "Resolves TODO in [file]"

### When Deferring TODOs

MUST: Leave TODO in place unchanged
MAY: Add clarifying comment if you learned relevant context
MUST NOT: Modify TODO text without implementing it
```

---

## Example 5: Response Format Specification

### Before (Human-Style)

```markdown
When explaining errors, be clear and helpful. Show what went wrong and
how to fix it. Include relevant code snippets if useful.
```

### After (Claude-Optimized)

```markdown
## Error Explanation Format

### Required Sections

```markdown
## Error: [Short description]

**What happened:**
[One sentence describing the failure]

**Why it happened:**
[Technical cause, 1-2 sentences]

**How to fix:**
[Numbered steps to resolve]

**Code change:**
[Before/after code snippet if applicable]
```

### Section Rules

**What happened:**
- MUST: Be one sentence
- MUST: Describe the symptom, not the cause
- MUST: Use plain language

**Why it happened:**
- MUST: Explain the technical cause
- MUST: Reference specific code location (file:line)
- SHOULD: Explain the mechanism (not just "it's wrong")

**How to fix:**
- MUST: Be numbered steps
- MUST: Be executable without additional research
- MUST: Start each step with imperative verb
- SHOULD: Be 3 or fewer steps

**Code change:**
- MUST: Show minimal change required
- MUST: Use diff format or before/after blocks
- MUST: Include file path
- MAY: Omit if fix is configuration, not code

### Example Output

```markdown
## Error: TypeError: Cannot read property 'id' of undefined

**What happened:**
The API returned a 500 error when fetching user profile.

**Why it happened:**
`req.user` is undefined in `src/controllers/users.ts:47` because
the auth middleware did not run before this route handler.

**How to fix:**
1. Open `src/routes/users.ts`
2. Add `authMiddleware` before the route handler on line 12
3. Run tests: `npm test src/routes/users.test.ts`

**Code change:**
```diff
// src/routes/users.ts:12
- router.get('/profile', userController.getProfile);
+ router.get('/profile', authMiddleware, userController.getProfile);
```
```

---

## Pattern Summary

### Transformation Techniques

| From | To |
|------|-----|
| "Try to..." | MUST: / SHOULD: / MAY: |
| "Consider..." | IF [condition]: THEN [action] |
| "Important" | MUST (non-negotiable) vs SHOULD (preferred) |
| "Appropriate" | [Enumerate the options explicitly] |
| "Use judgment" | Decision tree with conditions |
| "Be thorough" | Checklist of specific items |
| "When relevant" | IF [specific condition] |
| "Good practice" | SHOULD: [specific action] BECAUSE [reason] |

### Checklist for Transformation

1. **Find hedge words** → Replace with MUST/SHOULD/MAY
2. **Find undefined terms** → Add to Definitions section
3. **Find "use judgment"** → Create decision tree
4. **Find "be [adjective]"** → Convert to specific actions
5. **Find implicit knowledge** → Make explicit with conditions
6. **Find missing edge cases** → Enumerate with handling

---

**The transformation test:** Can another Claude session follow these instructions and produce identical behavior? If not, keep refining.
