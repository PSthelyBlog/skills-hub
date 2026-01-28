# Write-For-Claude Skill

---
name: write-for-claude
description: "Generate Claude-optimized instructions — unambiguous content that future Claude sessions parse identically"
version: 1.1.0
author: PSthelyBlog
tags: [meta, authoring, prompts, instructions, specifications, claude-md]
model: sonnet
examples:
  - "/write-for-claude create a CLAUDE.md for this project"
  - "/write-for-claude draft an AGENTS.md for a code review workflow"
  - "/write-for-claude write a skill spec for automated testing"
  - "/write-for-claude convert these notes into Claude-readable instructions"
---

## Trigger

WHEN user invokes `/write-for-claude [task]`: Execute this protocol.

## Protocol

### Step 1: Classify Output Type

Identify the document type from user request:

| IF request mentions | THEN output type is | REQUIRED sections |
|---------------------|---------------------|-------------------|
| CLAUDE.md, project instructions | `CLAUDE.md` | Context, Directives, Definitions, Anti-Patterns |
| SKILL.md, skill spec | `SKILL.md` | Frontmatter, Trigger, Protocol, Output Format, Edge Cases |
| AGENTS.md, agent definition | `AGENTS.md` | Role, Responsibilities, Inputs, Outputs, Handoffs |
| prompt, instructions | `Prompt` | Task, Constraints, Format |
| convert, transform | `Conversion` | Original → Transformed with changelog |

IF type unclear: Ask user to specify.

### Step 2: Transform Language

Apply these substitutions to all generated content:

#### 2.1 Replace Suggestions with Directives

| FIND | REPLACE WITH |
|------|--------------|
| Consider doing X | MUST: Do X when [state condition] |
| Try to X | SHOULD: X |
| You might want to X | MAY: X when [state condition] |
| It's important to X | MUST: X |
| Generally, X | SHOULD: X unless [state exception] |
| X is recommended | SHOULD: X |
| Avoid X | MUST NOT: X |
| X can help with Y | WHEN Y: do X |

#### 2.2 Eliminate Hedge Words

MUST NOT use these words in output:
- might, could, may (as uncertainty — MAY as permission is allowed)
- generally, usually, often, sometimes
- probably, likely, perhaps, maybe
- fairly, somewhat, relatively
- try to, consider, think about

REPLACE WITH: always, never, when [condition], if [condition], MUST, SHOULD, MAY

#### 2.3 Convert Implicit to Explicit

| FIND | REPLACE WITH |
|------|--------------|
| Use good judgment | IF [condition A]: do X. IF [condition B]: do Y. ELSE: do Z |
| Handle appropriately | [Enumerate each case with specific handling] |
| When relevant | WHEN [specific trigger]: |
| As needed | IF [condition]: do X. ELSE: skip |
| Important functionality | [List specific items that qualify] |

#### 2.4 Add Missing Conditions

FOR each directive in output:
  IF directive lacks trigger condition:
    ADD "WHEN [condition]:" prefix
    OR ADD "FOR [scope]:" prefix

FOR each "don't" rule:
  MUST: Include what TO do instead

### Step 3: Apply Structure

#### 3.1 Use Priority Markers

```
MUST: Non-negotiable. Violations are errors. Use for requirements.
SHOULD: Strong preference. Override requires explicit justification.
MAY: Optional. Include when beneficial.
MUST NOT: Prohibited. Never do this. Include reason.
```

MUST: Use these exact markers (not REQUIRED, SHALL, etc.)
MUST: One directive per line
MUST NOT: Combine multiple directives in prose paragraph

#### 3.2 Lead with Directive

```
WRONG:
Because performance matters and we want fast responses,
you should batch database queries when possible.

RIGHT:
SHOULD: Batch database queries when fetching >3 related records
RATIONALE: Reduces round trips, improves response time
```

MUST: State directive first
MAY: Add RATIONALE line after if helpful
MUST NOT: Bury directive in explanatory prose

#### 3.3 Enumerate Edge Cases

```
WRONG:
Handle errors gracefully.

RIGHT:
## Error Handling

WHEN network timeout: Retry once after 2s, then return error code TIMEOUT
WHEN invalid input: Return 400 with field-level validation messages
WHEN auth failure: Return 401, log attempt, do not retry
WHEN rate limited: Read retry-after header, wait, retry once
WHEN unknown error: Log full context with request ID, return 500
DEFAULT: Treat as unknown error
```

MUST: Include DEFAULT case for enumerations
MUST: Specify exact behavior, not adjectives ("gracefully")

#### 3.4 Create Decision Trees

```
WRONG:
Use your judgment about when to add tests.

RIGHT:
## Testing Decision Tree

IF function handles user input: MUST write test
ELSE IF function has >3 conditional branches: MUST write test
ELSE IF function calls external API: MUST write test
ELSE IF function is public export: SHOULD write test
ELSE: MAY write test
```

MUST: Include ELSE clause
MUST: Use IF/ELSE IF/ELSE structure (not prose conditions)
MUST: Each branch specifies action with priority marker

#### 3.5 Define Ambiguous Terms

IF output contains potentially ambiguous terms:
  ADD Definitions section

```
## Definitions

CRITICAL PATH: Authentication, payment processing, data persistence operations
CLEAN CODE: Functions <40 lines, single responsibility, descriptive names
PUBLIC FUNCTION: Exported from module, callable by other modules
HOT PATH: Code executed >1000 times per second under normal load
```

MUST: Define any term that could have multiple interpretations
MUST: Use measurable criteria where possible

### Step 4: Format by Output Type

#### 4.1 CLAUDE.md Format

```markdown
# [Project Name]

## Context

[One paragraph: what, who, purpose — max 3 sentences]

## Directives

### [Category 1]

MUST: [directive]
SHOULD: [directive]
MAY: [directive]

### [Category 2]

[Continue pattern]

## Definitions

[TERM]: [Measurable definition]

## Anti-Patterns

MUST NOT: [prohibited action]
INSTEAD: [correct action]

## Decision Trees

### [Decision Name]

IF [condition]: [action]
ELSE IF [condition]: [action]
ELSE: [default action]
```

#### 4.2 SKILL.md Format

```markdown
# [Skill Name]

---
name: [lowercase-with-hyphens]
description: "[action verb] [object] — [benefit]"
version: [semver]
model: [haiku|sonnet]
examples:
  - "[example invocation 1]"
  - "[example invocation 2]"
---

## Trigger

WHEN [condition]: Execute this protocol.

## Protocol

### Step 1: [Action Verb]

[Explicit instructions with IF/THEN/ELSE]

### Step 2: [Action Verb]

[Continue pattern]

## Output Format

```
[Exact template to follow]
```

## Edge Cases

WHEN [unusual condition 1]: [specific handling]
WHEN [unusual condition 2]: [specific handling]
DEFAULT: [fallback behavior]
```

#### 4.3 AGENTS.md Format

```markdown
## Agent: [Name]

### Role

[One sentence: verb phrase describing function]

### Scope

MUST: [what agent handles]
MUST NOT: [boundaries — what agent does not handle]

### Responsibilities

MUST: [primary duty 1]
MUST: [primary duty 2]
SHOULD: [secondary duty]
MAY: [optional duty]

### Inputs

- [Input 1]: [description, format]
- [Input 2]: [description, format]

### Output Format

```
[Exact structure]
```

### Handoff Conditions

WHEN [condition 1]: Hand off to [Agent Name]
WHEN [condition 2]: Hand off to [Agent Name]
DEFAULT: Complete task, return output
```

### Step 5: Validate Output

BEFORE returning output, verify ALL of:

- [ ] Every MUST/SHOULD/MAY has explicit trigger (WHEN/IF/FOR) or applies universally
- [ ] Zero hedge words present (search for: might, could, generally, try to, consider)
- [ ] Every MUST NOT has corresponding correct action (INSTEAD)
- [ ] Definitions section exists IF output contains terms with multiple interpretations
- [ ] Decision trees have ELSE/DEFAULT clauses
- [ ] Enumerations have DEFAULT case
- [ ] Examples show edge cases, not only happy paths
- [ ] No instruction requires "judgment" without criteria for that judgment

IF validation fails: Fix issues before returning.

### Step 6: Return Output

Return generated document followed by:

```markdown
---

## Optimization Applied

CONVERTED TO EXPLICIT:
- [what vague term] → [specific measurable criteria]

ADDED TRIGGERS:
- [directive] now specifies WHEN [condition]

DEFINED TERMS:
- [term]: [definition added]

ADDED EDGE CASES:
- [scenario]: [handling specified]
```

## Edge Cases

WHEN user provides well-structured input that already follows principles:
  Validate against checklist, return "Input already follows Claude-optimization principles" if passes, or fix specific gaps if not.

WHEN user request is ambiguous about output type:
  Ask: "What type of document? (CLAUDE.md / SKILL.md / AGENTS.md / Prompt)"

WHEN user provides very short input with minimal context:
  Generate structure with placeholder markers: `[SPECIFY: what information needed]`

WHEN user asks to convert content containing genuinely subjective decisions:
  Convert to decision tree with explicit criteria; note where human judgment is required: `REQUIRES HUMAN INPUT: [decision]`

WHEN output would exceed reasonable length (>500 lines):
  Split into sections, offer: "Generated [Section 1]. Continue with [Section 2]?"

## Validation Test

FOR generated output, apply this test:

```
Test: Could another Claude session, reading ONLY this output
(no prior context), produce identical behavior?

IF yes: Output passes
IF no: Identify ambiguity, make explicit, re-validate
```

## Anti-Patterns in This Skill's Output

MUST NOT generate:

```markdown
// WRONG: Prose explanation
When working with this codebase, it's important to maintain
clean code practices and ensure proper test coverage. Consider
the existing patterns before adding new abstractions.

// WRONG: Vague directive
SHOULD: Write good tests

// WRONG: Missing trigger
MUST: Use TypeScript strict mode
(When? For all files? New files only?)

// WRONG: No alternative
MUST NOT: Use `any` type
(What should they use instead?)

// WRONG: Implicit judgment
Handle edge cases appropriately
(Which edge cases? What's appropriate?)

// WRONG: Missing default
IF user input invalid: Return 400
IF auth fails: Return 401
(What about other errors?)
```

MUST generate:

```markdown
// RIGHT: Structured directives
## Code Style

MUST: Use TypeScript strict mode for all files in src/
MUST NOT: Use `any` type
  INSTEAD: Use `unknown` with type guards, or define specific type

## Testing

MUST: Write unit tests for functions that:
  - Handle user input
  - Have >3 conditional branches
  - Call external APIs
SHOULD: Write integration tests for new API endpoints
MAY: Add property-based tests for pure transformation functions

## Error Handling

WHEN input validation fails: Return 400 with `{ field, message }` array
WHEN authentication fails: Return 401 with `{ error: "Unauthorized" }`
WHEN authorization fails: Return 403 with `{ error: "Forbidden" }`
WHEN resource not found: Return 404 with `{ error: "Not found" }`
WHEN rate limited: Return 429 with `{ retryAfter: seconds }`
DEFAULT: Return 500 with `{ error: "Internal error", requestId }`

## Definitions

CLEAN CODE: Functions <40 lines, single responsibility, no duplication >3 lines
PROPER TEST COVERAGE: >80% line coverage for new modules
EXISTING PATTERN: Implementation used in 2+ files in same directory
```

---

## Meta-Compliance

This specification follows its own rules:
- Uses MUST/SHOULD/MAY throughout
- Every directive has trigger or universal scope
- Decision trees have ELSE/DEFAULT
- Terms defined in context
- Edge cases enumerated
- No hedge words
- Examples show correct vs incorrect

Validation test result: Another Claude session reading this specification will produce identical behavior.
