# Suggest Skill

---
name: suggest
description: "Analyze code and recommend actionable improvements with before/after diffs"
version: 1.1.0
author: PSthelyBlog
tags: [refactoring, code-quality, improvements, review]
model: haiku
examples:
  - "/suggest"
  - "/suggest src/auth.ts"
  - "/suggest --category performance"
  - "/suggest --depth thorough --limit 10"
  - "/suggest --apply"
---

## Purpose

The `/suggest` skill complements `/explain` by answering **"how could this be better?"** after explain answers **"what does this do?"**

Where `/explain` builds understanding, `/suggest` drives improvement. Together they form a complete code review workflow.

## Core Philosophy

**Curated beats comprehensive.**

By default, suggestions are:
- **Actionable** - Every suggestion includes before/after code
- **Prioritized** - Ranked by impact, limited to 5 by default
- **Contextual** - Understands the code from prior `/explain` if available

## How It Works

1. **Analyzes the target code** - Reads file, function, or selection
2. **Identifies improvement opportunities** - Bugs, performance, readability, security
3. **Ranks by impact** - High-impact, low-effort suggestions first
4. **Presents with diffs** - Before/after code for each suggestion
5. **Optionally applies** - Interactive mode to implement changes

## Instructions for Claude

When the user invokes `/suggest [target]`, follow this protocol:

### 1. Parse the Request

Extract:
- **Target**: File path, function name, or use current selection/context
- **Category filter**: If `--category` flag present (bugs, security, performance, readability)
- **Depth level**: If `--depth` flag present (quick, standard, thorough)
- **Limit**: If `--limit N` flag present (default: 5, max: 10)
- **Apply mode**: If `--apply` flag present

### 2. Gather Context

**If following `/explain`:**
- Reuse the context and understanding from the previous explanation
- Reference insights already surfaced
- Don't re-read files unnecessarily

**If standalone:**
- Read the target file(s)
- Understand the code's purpose and patterns
- Identify the language and framework conventions

### 3. Analyze for Improvements

Scan for issues in these categories:

| Category | What to Look For |
|----------|------------------|
| **bugs** | Potential runtime errors, edge cases, null checks, type issues, unhandled promises |
| **security** | Input validation, injection risks, auth gaps, secrets exposure, OWASP top 10 |
| **performance** | N+1 queries, unnecessary loops, missing memoization, blocking calls, large bundles |
| **readability** | Long functions, unclear naming, missing abstractions, code duplication, magic numbers |

**Depth levels:**

- `quick` - Surface-level patterns only (lint-style checks, obvious issues)
- `standard` - Common issues + logical improvements (default)
- `thorough` - Deep analysis including architectural suggestions, cross-file patterns

### 4. Rank and Filter

1. Score each issue by **impact** (how much it matters) and **effort** (how hard to fix)
2. Prioritize: HIGH impact + LOW effort first
3. Apply category filter if specified with `--category`
4. Limit to N suggestions (default 5, respect `--limit` flag)

**Severity assignment:**
- `[HIGH]` - Bug, security vulnerability, crash risk, data loss potential
- `[MED]` - Performance issue, maintainability problem, code smell
- `[LOW]` - Style improvement, minor optimization, nice-to-have

### 5. Format Output

Produce this structured output:

```markdown
## Suggestions for [target]

### 1. [SEVERITY] Brief descriptive title
**Category:** category-name
**Why:** One sentence explaining the benefit of this change

**Before:**
```language
// The problematic code exactly as it appears
```

**After:**
```language
// The improved code with the fix applied
```

---

### 2. [SEVERITY] Brief descriptive title
**Category:** category-name
**Why:** One sentence explaining the benefit

**Before:**
```language
// Code before
```

**After:**
```language
// Code after
```

---

[Continue for each suggestion up to limit]

**Summary:** N suggestions (X high, Y medium, Z low)
Run `/suggest --apply` to interactively apply these changes.
```

### 6. Apply Mode Protocol

If `--apply` flag is present:

1. **Present each suggestion one at a time**:
   ```
   Applying suggestion 1/N: [Title]

   **Before:**
   [show the exact code to be replaced]

   **After:**
   [show the replacement code]

   Apply this change? (y/n/skip all)
   ```

2. **Wait for user response** using AskUserQuestion tool

3. **If approved**: Use Edit tool to apply the change

4. **Continue to next suggestion** or skip remaining if user chooses

5. **Summarize at end**:
   ```
   Applied 3/5 suggestions:
   - [Title 1] ✓
   - [Title 2] ✓
   - [Title 3] (skipped)
   - [Title 4] ✓
   - [Title 5] (skipped)
   ```

### 7. Chaining with /explain

The suggest skill works seamlessly after explain:

```
User: /explain src/auth.ts
Claude: [explains the authentication module]

User: /suggest
Claude: [suggests improvements, using same target and context]

User: /suggest --apply
Claude: [interactively applies selected improvements]
```

**When no target is specified after `/explain`**: Automatically use the same target from the previous explain.

### 8. Example Interactions

**Example 1: Basic suggestion**
```
User: /suggest src/utils/format.ts

Claude:
## Suggestions for src/utils/format.ts

### 1. [HIGH] Add null check before string operations
**Category:** bugs
**Why:** Prevents TypeError when input is undefined

**Before:**
```typescript
function formatName(name: string) {
  return name.trim().toLowerCase();
}
```

**After:**
```typescript
function formatName(name: string | null | undefined) {
  if (!name) return '';
  return name.trim().toLowerCase();
}
```

---

### 2. [MED] Extract repeated date formatting logic
**Category:** readability
**Why:** Reduces duplication and centralizes format changes

**Before:**
```typescript
const date1 = `${d.getFullYear()}-${d.getMonth()+1}-${d.getDate()}`;
const date2 = `${d2.getFullYear()}-${d2.getMonth()+1}-${d2.getDate()}`;
```

**After:**
```typescript
const formatDate = (d: Date) =>
  `${d.getFullYear()}-${String(d.getMonth()+1).padStart(2,'0')}-${String(d.getDate()).padStart(2,'0')}`;

const date1 = formatDate(d);
const date2 = formatDate(d2);
```

---

**Summary:** 2 suggestions (1 high, 1 medium)
Run `/suggest --apply` to interactively apply these changes.
```

**Example 2: Category filter**
```
User: /suggest --category security src/api/

Claude:
## Suggestions for src/api/ (security)

### 1. [HIGH] Sanitize user input before SQL query
**Category:** security
**Why:** Prevents SQL injection attacks

**Before:**
```typescript
const query = `SELECT * FROM users WHERE id = ${userId}`;
```

**After:**
```typescript
const query = 'SELECT * FROM users WHERE id = $1';
const result = await db.query(query, [userId]);
```

---

[... more security-focused suggestions ...]
```

**Example 3: Apply mode**
```
User: /suggest --apply src/auth.ts

Claude:
Applying suggestion 1/3: Add error handling for token verification

**Before:**
```typescript
const decoded = jwt.verify(token, SECRET);
```

**After:**
```typescript
try {
  const decoded = jwt.verify(token, SECRET);
} catch (error) {
  throw new AuthenticationError('Invalid token');
}
```

Apply this change? (y/n/skip all)

User: y

Claude: Applied. Moving to suggestion 2/3...

[... continues through suggestions ...]

Applied 2/3 suggestions:
- Add error handling for token verification ✓
- Use constant-time comparison for secrets ✓
- Extract token refresh logic (skipped)
```

### 9. Error Cases

**If target not found:**
```
Could not find [target]. Did you mean:
- src/utils/format.ts
- src/helpers/formatter.js

Or specify a different path.
```

**If no improvements found:**
```
## Suggestions for [target]

No significant improvements found. The code follows good practices for:
- Error handling ✓
- Input validation ✓
- Performance patterns ✓
- Readability ✓

Consider running with `--depth thorough` for deeper analysis.
```

**If file too large:**
```
[target] is large (2000+ lines). For better results:
- Target a specific function: `/suggest functionName`
- Use category filter: `/suggest --category bugs [target]`
- Limit scope: `/suggest [target]:1-100` (lines 1-100)
```

### 10. Context Awareness

Before suggesting, consider:
- **Language/Framework**: TypeScript vs JavaScript, React vs Vue, etc.
- **Project conventions**: Existing patterns, style guide, linting rules
- **Test coverage**: Don't break existing tests
- **Dependencies**: Suggest using existing utilities over new ones

Tailor suggestions to match the codebase's existing style.

## Flags Reference

| Flag | Description | Default |
|------|-------------|---------|
| `--category` | Filter: `bugs`, `security`, `performance`, `readability` | all |
| `--depth` | Analysis depth: `quick`, `standard`, `thorough` | standard |
| `--limit N` | Maximum suggestions (1-10) | 5 |
| `--apply` | Interactively apply suggestions | off |
| `--no-diffs` | Hide before/after code blocks | show diffs |
| `--json` | Output as JSON for tooling integration | off |

## Usage Patterns

### After Understanding Code
```bash
/explain src/api/users.ts
/suggest                      # Uses same context
```

### Targeted Improvements
```bash
/suggest --category security src/auth/
/suggest --category performance --depth thorough
```

### Quick Wins
```bash
/suggest --depth quick --limit 3    # Fast, actionable items
```

### Full Review
```bash
/suggest --depth thorough --limit 10 src/
```

### Apply Interactively
```bash
/suggest --apply src/utils.ts
```

## Success Metrics

A good suggestion:
- Has clear before/after code that can be directly applied
- Explains *why* it's an improvement in one sentence
- Is immediately actionable without major refactoring
- Doesn't require extensive context to understand

A great suggestion:
- Catches a bug the developer would have missed
- Teaches a better pattern for future code
- Applies cleanly with `--apply` mode
- Makes the developer think "why didn't I do that?"

## Notes for Skill Authors

This skill synthesizes three perspectives:

1. **Pragmatist contribution**: Limited suggestions (5 default), quick depth option, immediate applicability
2. **Perfectionist contribution**: Severity levels, category filtering, thorough depth option, structured format
3. **User Advocate contribution**: Before/after diffs, interactive apply mode, clear one-sentence explanations

The goal isn't to find everything wrong—it's to surface the improvements that matter most, right now.

---

**Remember**: Explain tells you what code does. Suggest tells you what code could become. The power is in the pairing.
