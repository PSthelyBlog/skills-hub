# Suggest Skill

---
name: suggest
description: "Analyze code and recommend actionable improvements with before/after diffs"
version: 1.2.0
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

## Trigger

WHEN user invokes `/suggest [target]`: Execute this protocol.
WHEN user invokes `/suggest` after `/explain`: Use same target from explain.

## Protocol

### Step 1: Parse Request

Extract from user input:

| Component | Flag | Default | Valid values |
|-----------|------|---------|--------------|
| Target | positional | Current file or prior explain target | File path, function name, directory |
| Category | `--category` | all | bugs, security, performance, readability |
| Depth | `--depth` | standard | quick, standard, thorough |
| Limit | `--limit N` | 5 | 1-10 |
| Apply mode | `--apply` | off | on/off |
| Show diffs | `--no-diffs` | show | show/hide |
| JSON output | `--json` | off | on/off |

IF no target specified AND prior `/explain` exists in conversation:
  Use target from prior explain
IF no target specified AND no prior explain:
  Use current file context
IF no current file context:
  Ask: "What file or function should I analyze?"

### Step 2: Gather Context

#### 2.1 Context Decision Tree

```
IF prior /explain exists for same target:
  Reuse context and understanding from explain
  MUST NOT: Re-read files already analyzed
ELSE:
  Read target file(s)
  Identify language and framework
  Note existing patterns and conventions
```

#### 2.2 Context Requirements

MUST: Identify language (for syntax in diffs)
MUST: Note existing code style (match in suggestions)
MUST: Check for existing tests (avoid breaking changes)
SHOULD: Identify project dependencies (suggest existing utilities)

### Step 3: Analyze for Improvements

#### 3.1 Category Definitions

| Category | Scan for |
|----------|----------|
| bugs | Runtime errors, edge cases, null checks, type issues, unhandled promises, race conditions |
| security | Input validation, injection risks, auth gaps, secrets exposure, OWASP top 10 |
| performance | N+1 queries, unnecessary loops, missing memoization, blocking calls, large bundles |
| readability | Long functions (>40 lines), unclear naming, code duplication (>3 lines), magic numbers |

#### 3.2 Depth Levels

| Depth | Scope | Use when |
|-------|-------|----------|
| quick | Surface patterns only (lint-style, obvious issues) | Fast feedback, small changes |
| standard | Common issues + logical improvements | Default analysis |
| thorough | Deep analysis, architectural suggestions, cross-file patterns | Full review, pre-merge |

IF `--category` specified: Scan only that category
ELSE: Scan all categories

### Step 4: Rank and Filter

#### 4.1 Severity Assignment

| Severity | Assign when |
|----------|-------------|
| HIGH | Bug, security vulnerability, crash risk, data loss potential |
| MED | Performance issue, maintainability problem, code smell |
| LOW | Style improvement, minor optimization, nice-to-have |

#### 4.2 Ranking Rules

MUST: Score each issue by impact (how much it matters) × effort (inverse of difficulty)
MUST: Sort by score descending (HIGH impact + LOW effort first)
MUST: Apply category filter if `--category` specified
MUST: Limit to N suggestions (respect `--limit` flag, default 5, max 10)

### Step 5: Format Output

#### 5.1 Standard Output Format

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

#### 5.2 Format Rules

MUST: Include severity in brackets: `[HIGH]`, `[MED]`, `[LOW]`
MUST: Include category label
MUST: Include one-sentence "Why" explanation
MUST: Show Before/After code blocks with correct language tag
MUST: Use exact code from file in Before block (not paraphrased)
MUST: End with summary count and `--apply` hint
MUST NOT: Include Before/After if `--no-diffs` flag present

IF `--json` flag present:
  Output as JSON array with fields: `severity`, `category`, `title`, `why`, `before`, `after`, `line_start`, `line_end`

### Step 6: Apply Mode

WHEN `--apply` flag present:

#### 6.1 Apply Protocol

FOR each suggestion (in order):

```
1. Present suggestion:
   "Applying suggestion N/M: [Title]

   **Before:**
   [exact code to replace]

   **After:**
   [replacement code]

   Apply this change?"

2. Use AskUserQuestion with options:
   - "Yes, apply"
   - "Skip this one"
   - "Skip all remaining"

3. IF "Yes, apply": Use Edit tool to apply change
   IF "Skip this one": Continue to next
   IF "Skip all remaining": Exit loop

4. Continue until all suggestions processed or user skips all
```

#### 6.2 Apply Summary

AFTER all suggestions processed:

```markdown
Applied X/N suggestions:
- [Title 1] ✓
- [Title 2] ✓
- [Title 3] (skipped)
- [Title 4] ✓
```

### Step 7: Handle Errors

WHEN target not found:

```markdown
Could not find [target]. Did you mean:
- [similar file 1]
- [similar file 2]

Or specify a different path.
```

MUST: Use Glob to find similar files

WHEN no improvements found:

```markdown
## Suggestions for [target]

No significant improvements found at `--depth [current depth]`. The code follows good practices.

Options:
- Run with `--depth thorough` for deeper analysis
- Try `--category [specific]` for focused review
```

WHEN file too large (>1000 lines):

```markdown
[target] is large ([N] lines). For better results:
- Target a specific function: `/suggest functionName`
- Use category filter: `/suggest --category bugs [target]`
- Limit scope: `/suggest [target]:1-200` (lines 1-200)
```

## Definitions

SUGGESTION: A specific, actionable code change with before/after examples
SEVERITY: Impact level of the issue (HIGH/MED/LOW)
CATEGORY: Type of improvement (bugs/security/performance/readability)
DEPTH: How thoroughly to analyze (quick/standard/thorough)
APPLY MODE: Interactive mode where user approves each change before it's made

## Edge Cases

WHEN suggestion would break existing tests:
  MUST: Note in suggestion: "May require test updates"
  SHOULD: Show what test changes might be needed

WHEN suggestion conflicts with project style:
  MUST: Match existing project conventions
  MUST NOT: Suggest style changes that contradict codebase patterns

WHEN multiple suggestions affect same code block:
  MUST: Present as single combined suggestion
  MUST NOT: Present overlapping suggestions separately

WHEN target is a directory:
  MUST: Analyze all supported files in directory
  MUST: Group suggestions by file
  SHOULD: Limit to most impactful suggestions across all files

WHEN prior explain context exists but target differs:
  MUST: Treat as standalone (do not reuse explain context)
  MUST: Read new target fresh

DEFAULT: If uncertain about severity, assign MED.

## Anti-Patterns

MUST NOT: Suggest changes without before/after code

```markdown
// WRONG: Vague suggestion
### 1. [MED] Improve error handling
**Why:** Better error handling would help
[No code shown]

// RIGHT: Concrete with diff
### 1. [MED] Add try-catch around API call
**Why:** Prevents unhandled promise rejection from crashing server

**Before:**
```typescript
const data = await fetch(url).then(r => r.json());
```

**After:**
```typescript
try {
  const data = await fetch(url).then(r => r.json());
} catch (error) {
  logger.error('API fetch failed', { url, error });
  throw new ApiError('Failed to fetch data');
}
```
```

MUST NOT: Suggest style-only changes as HIGH severity

```markdown
// WRONG: Style as HIGH
### 1. [HIGH] Use const instead of let
**Category:** readability

// RIGHT: Style as LOW
### 1. [LOW] Use const instead of let
**Category:** readability
```

MUST NOT: Suggest breaking changes without warning

```markdown
// WRONG: Breaking change unmarked
### 1. [MED] Change function signature
**After:**
function getUser(id: string, options: Options) // was (id: string)

// RIGHT: Breaking change flagged
### 1. [MED] Change function signature
**Why:** Adds flexibility for future options
**Breaking:** Callers must be updated to pass options parameter
```

MUST NOT: Show paraphrased code in Before block

```markdown
// WRONG: Paraphrased
**Before:**
```typescript
// some date formatting code here
```

// RIGHT: Exact code from file
**Before:**
```typescript
const formatted = `${d.getFullYear()}-${d.getMonth()+1}-${d.getDate()}`;
```
```

## Flags Reference

| Flag | Description | Default |
|------|-------------|---------|
| `--category` | Filter: bugs, security, performance, readability | all |
| `--depth` | Analysis depth: quick, standard, thorough | standard |
| `--limit N` | Maximum suggestions (1-10) | 5 |
| `--apply` | Interactively apply suggestions | off |
| `--no-diffs` | Hide before/after code blocks | show diffs |
| `--json` | Output as JSON for tooling integration | off |

## Validation

BEFORE returning, verify:

- [ ] Target file was read (or context reused from explain)
- [ ] Each suggestion has severity label
- [ ] Each suggestion has category label
- [ ] Each suggestion has one-sentence "Why"
- [ ] Before block contains exact code from file
- [ ] After block contains valid replacement code
- [ ] Suggestions sorted by impact (HIGH first)
- [ ] Suggestion count respects --limit
- [ ] Summary line present with counts
- [ ] --apply hint present (unless in apply mode)

IF validation fails: Fix before returning.
