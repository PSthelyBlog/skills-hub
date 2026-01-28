# Write-For-Claude Skill

**Generate instructions that future Claude sessions parse reliably.**

## The Problem

When Claude writes CLAUDE.md files, skill specifications, or agent definitions, it defaults to human documentation patterns. The result is content that reads well but interprets ambiguously.

```markdown
// Human-style (ambiguous)
Try to keep code simple and consider existing patterns.
Tests are important, especially for critical functionality.

// Claude-optimized (unambiguous)
MUST: Match patterns in same directory before creating new ones
MUST: Write tests for functions handling user input
MUST: Write tests for functions with >3 conditional branches
```

The `/write-for-claude` skill teaches Claude to generate machine-parseable content from the start.

## Quick Start

```bash
# Generate a CLAUDE.md for your project
/write-for-claude create a CLAUDE.md for this Python web API project

# Convert existing notes to Claude instructions
/write-for-claude convert these guidelines to unambiguous directives

# Create an agent definition
/write-for-claude define an agent for code review

# Write a skill specification
/write-for-claude create a skill for formatting JSON
```

## What It Does

1. **Eliminates ambiguity** - Converts suggestions into explicit directives
2. **Adds structure** - Uses MUST/SHOULD/MAY priority markers
3. **Covers edge cases** - Enumerates conditions instead of "use judgment"
4. **Defines terms** - Makes potentially ambiguous words explicit
5. **Validates output** - Runs anti-ambiguity checks before delivering

## The Transformation

### Before (Human-Style)

```markdown
When modifying code, try to follow existing conventions.
Keep things clean and well-tested. Use your judgment about
what needs documentation. Performance matters but don't
over-optimize prematurely.
```

### After (Claude-Optimized)

```markdown
## Directives

### Code Style
MUST: Match indentation and formatting of surrounding code
MUST: Use existing utility functions from src/utils/ if available
SHOULD: Keep functions under 40 lines
SHOULD: Prefer composition over inheritance

### Testing
MUST: Write tests for new public functions
MUST: Run existing test suite before committing
SHOULD: Add edge case tests when fixing bugs
MAY: Add performance benchmarks for hot paths

### Documentation
MUST: Add JSDoc for public API functions
SHOULD: Add inline comments for non-obvious algorithms
MAY: Add README updates for new features

### Performance
MUST: Profile before optimizing
MUST NOT: Optimize without measured performance data
SHOULD: Prefer readability over micro-optimizations
EXCEPTION: For functions called >1000x/second, prioritize performance

## Definitions

EXISTING CONVENTION: Pattern used in 2+ files in same directory
PUBLIC FUNCTION: Exported function callable from other modules
HOT PATH: Code executed >1000 times per second under load
```

## Key Principles

### 1. Imperatives Over Suggestions

| Don't Write | Write Instead |
|-------------|---------------|
| Consider adding tests | MUST: Add tests for [specific scope] |
| Try to keep it simple | SHOULD: Keep functions under N lines |
| You might want to... | When [condition], do [action] |
| Generally, X is good | Do X unless [specific exception] |

### 2. Explicit Conditions

| Don't Write | Write Instead |
|-------------|---------------|
| Use good judgment | IF [condition A]: do X. IF [condition B]: do Y |
| Handle errors appropriately | List each error type and its handling |
| Test critical paths | Define "critical" + list specific paths |

### 3. Priority Markers

```markdown
MUST: Non-negotiable. Violations are errors.
SHOULD: Strong preference. Override only with explicit justification.
MAY: Optional. Use when beneficial.
MUST NOT: Prohibited. Never do this.
```

### 4. Complete Edge Cases

```markdown
// Don't write:
Handle errors gracefully.

// Write:
ERROR HANDLING:
- Network timeout: Retry once after 2s, then return error with code TIMEOUT
- Invalid input: Return 400 with field-level validation messages
- Auth failure: Return 401, do not retry, log attempt
- Rate limit: Wait for retry-after header value, then retry once
- Unknown error: Log full context, return 500 with request ID
- DEFAULT: If error type unclear, treat as unknown error
```

## Use Cases

### Creating CLAUDE.md Files

```bash
/write-for-claude create CLAUDE.md for this TypeScript monorepo
```

Generates project instructions with:
- Explicit code style directives
- Testing requirements with specific scope
- Architecture patterns to follow
- Anti-patterns to avoid
- Defined terminology

### Writing Skill Specifications

```bash
/write-for-claude write a skill spec for automated PR descriptions
```

Generates SKILL.md with:
- Step-by-step protocol (not prose)
- Exact output format
- Edge case handling
- Example inputs and outputs

### Defining Agents

```bash
/write-for-claude define a security review agent
```

Generates AGENTS.md section with:
- Explicit responsibilities (MUST/MAY/MUST NOT)
- Input/output specifications
- Handoff conditions
- Severity definitions

### Converting Existing Documentation

```bash
/write-for-claude convert this style guide to Claude directives
```

Transforms human documentation into:
- Actionable directives
- Explicit priorities
- Defined terms
- Enumerated edge cases

## The Validation Checklist

Before outputting, the skill verifies:

- [ ] Every instruction has a clear trigger condition
- [ ] No hedge words (might, could, generally, try to)
- [ ] All "don't" rules have "do instead" alternatives
- [ ] Definitions section covers ambiguous terms
- [ ] Examples include edge cases
- [ ] No instruction requires undefined "judgment"
- [ ] Decision trees have ELSE clauses
- [ ] Priorities explicit when rules could conflict

## Example Output

### Input Request

"Create instructions for a code review agent"

### Generated Output

```markdown
## Agent: Code Reviewer

### Role
Analyze pull requests for correctness, style, and maintainability.

### Responsibilities

MUST: Check for logical errors in changed code
MUST: Verify test coverage for new functions
MUST: Flag style violations per project conventions
SHOULD: Suggest performance improvements for O(n^2)+ operations
SHOULD: Note potential edge cases not covered by tests
MAY: Suggest refactoring opportunities in touched code

### Scope Boundaries

MUST NOT: Review files not modified in the PR
MUST NOT: Suggest changes to unrelated code areas
MUST NOT: Block PR for style-only issues (suggest, don't block)

### Review Checklist

For each changed file:
1. IF new function: verify test exists
2. IF modified function: verify existing tests still pass
3. IF public API change: verify documentation updated
4. IF error handling added: verify all paths covered

### Output Format

```markdown
## Code Review: [PR Title]

### Must Fix (Blocking)
- [file:line] [issue] [suggestion]

### Should Fix (Non-blocking)
- [file:line] [issue] [suggestion]

### Suggestions (Optional)
- [file:line] [opportunity]

### Approved Sections
- [List of reviewed areas with no issues]
```

### Handoff Conditions

WHEN security-related code: Flag for Security Reviewer agent
WHEN performance-critical path: Flag for Performance Reviewer agent
WHEN documentation-only PR: Skip to auto-approve
```

## Why This Matters

Human documentation optimizes for readability. Claude-consumable content must optimize for **unambiguous interpretation**.

A future Claude session reading your instructions has:
- No context beyond what you wrote
- No ability to ask clarifying questions
- No intuition about "what you probably meant"

Every ambiguity becomes a potential inconsistency across sessions.

The `/write-for-claude` skill produces content that:
- Behaves identically across Claude sessions
- Requires no inference or judgment calls
- Fails explicitly rather than ambiguously
- Can be followed literally and correctly

## Installation

```bash
cp -r write-for-claude ~/.claude/skills/
```

See [INSTALL.md](INSTALL.md) for details.

## Related

- [SKILL.md](SKILL.md) - Full specification
- [EXAMPLES.md](EXAMPLES.md) - Before/after transformations
- [INSTALL.md](INSTALL.md) - Installation guide

---

**The test:** Could another Claude session, reading only your instructions, behave identically? If not, use `/write-for-claude`.
