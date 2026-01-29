# Explain Skill

---
name: explain
description: "Transform codebase questions into progressive explanations — quick answers by default, deep analysis on request"
version: 1.2.0
author: PSthelyBlog
tags: [learning, documentation, codebase-understanding]
model: haiku
examples:
  - "/explain what does this function do"
  - "/explain authentication flow --deep"
  - "/explain why is the build failing"
  - "/explain dependency between users and posts"
---

## Trigger

WHEN user invokes `/explain [query]`: Execute this protocol.
WHEN user invokes `/explain [query] --deep`: Execute this protocol with deep analysis enabled.

## Protocol

### Step 1: Parse the Request

Extract from user input:

| Component | Source | Required |
|-----------|--------|----------|
| Subject | First non-flag argument | MUST be present |
| Depth flag | `--deep` anywhere in query | MAY be absent (defaults to quick mode) |
| Context | Current working directory, recently viewed files | Auto-captured |

IF subject missing: Ask "What would you like me to explain?"

### Step 2: Determine Scope

| IF condition | THEN mode | Model |
|--------------|-----------|-------|
| `--deep` flag present | Deep | Sonnet |
| Query contains "architecture", "design", "system", "flow" | Deep | Sonnet |
| Query contains "performance", "complexity", "scale" | Deep | Sonnet |
| Query asks "why" about design decisions | Deep | Sonnet |
| Query asks about cross-file dependencies | Deep | Sonnet |
| Single function or code block | Quick | Haiku |
| Error message lookup | Quick | Haiku |
| "What does X do" format | Quick | Haiku |
| DEFAULT | Quick | Haiku |

IF quick mode reveals unexpected complexity mid-explanation:
  Notify user: "This is more complex than it appeared — switching to deeper analysis."
  Continue with deep mode.

### Step 3: Gather Context

#### 3.1 Minimal Search (Always First)

MUST: Search for subject using Grep
MUST: Read only relevant sections (not full files)
MUST: Check for existing documentation in same directory
MUST NOT: Read entire files when searching for specific function

#### 3.2 Expanded Search (Deep Mode Only)

WHEN deep mode:
  MUST: Trace dependencies (imports, exports)
  MUST: Check related tests
  SHOULD: Review git history for recent changes to subject
  MAY: Check related modules for usage patterns

#### 3.3 Context Awareness

BEFORE explaining, check:

| Context | Purpose |
|---------|---------|
| Recent file reads in conversation | Tailor explanation to what user is viewing |
| Current directory (tests/src/docs) | Adjust terminology and focus |
| Git context (recent commits) | Understand what user is working on |

### Step 4: Generate Response

#### 4.1 Quick Mode Format

```markdown
## [Subject Name]

**What it does**: [One sentence — max 15 words]

**Key points**:
- [Essential fact 1]
- [Essential fact 2]
- [Essential fact 3]

**Why it matters**: [Practical implication for user's workflow]

---
Need more? Add `--deep` for full analysis.
```

MUST: Limit to 3-5 key points
MUST: Include "Why it matters" with actionable context
MUST: Offer `--deep` option at end

#### 4.2 Deep Mode Format

```markdown
## [Subject Name]

**Overview**: [2-3 sentence comprehensive summary]

**How it works**:
1. [Step with file:line reference]
2. [Step with file:line reference]
3. [Continue as needed]

**Dependencies**:
- Relies on: [list with file paths]
- Used by: [list with file paths]

**Design choices**:
- [Choice]: [Rationale]
- Trade-off: [What was gained] vs [what was sacrificed]

**Common patterns**:
[How this fits into broader codebase patterns]

**Gotchas**:
- [Edge case or known issue 1]
- [Edge case or known issue 2]

**Related**:
- `[file:line]` — [brief description]
- `[file:line]` — [brief description]
```

MUST: Include file:line references for all code mentions
MUST: Explain trade-offs in design choices
MUST: List at least one gotcha (or state "None identified")

### Step 5: Apply Teaching Principles

#### 5.1 Language Rules

MUST: Use plain language first, technical terms second
MUST: Show code examples from user's actual codebase
MUST: Connect new concepts to familiar ones
MUST: Highlight practical implications

MUST NOT: Dump full file contents
MUST NOT: Over-explain obvious constructs
MUST NOT: Give generic advice disconnected from user's code
MUST NOT: Assume expertise level without calibration from context

#### 5.2 Progressive Disclosure

AFTER quick explanation, offer ONE natural next step:

| IF user asked about | THEN offer |
|---------------------|------------|
| Function behavior | "Want to see how this handles errors?" |
| Data flow | "Curious about performance characteristics?" |
| Any subject | "Need to modify this? I can show related tests." |

MUST: Offer only one follow-up (not a list)
MUST: Phrase as question, not statement

### Step 6: Handle Errors

WHEN subject not found:

1. MUST: Search for similar matches
2. MUST: Suggest alternatives: "Did you mean [similar name]?"
3. MUST: Offer broader search: "I can search for all [category]-related code"
4. MAY: Ask for clarification: "Can you point me to the file?"

MUST NOT: Say "I can't find that" without offering next steps

WHEN multiple matches found:

1. MUST: List matches with file paths
2. MUST: Ask user to specify: "Which one? [list options]"

## Definitions

QUICK MODE: Explanation completed in <30 seconds reading time, focused on immediate practical value
DEEP MODE: Comprehensive analysis including dependencies, design rationale, and edge cases
SUBJECT: The function, file, concept, error, or pattern the user wants explained
GOTCHA: Edge case, known issue, or non-obvious behavior that could cause problems

## Edge Cases

WHEN user asks about deprecated code:
  MUST: Explain what it does
  MUST: Note deprecation status
  MUST: Point to replacement if one exists

WHEN user asks about third-party library internals:
  SHOULD: Explain how project uses the library
  MAY: Explain library internals if directly relevant
  MUST NOT: Deep-dive into external library source

WHEN user asks "why" without specific subject:
  Ask: "What specifically would you like me to explain the reasoning behind?"

WHEN explanation would exceed 500 words (quick mode):
  Truncate to key points
  Add: "This topic has more depth — add `--deep` for full analysis."

DEFAULT: Provide quick mode explanation with offer to go deeper.

## Anti-Patterns

MUST NOT generate:

```markdown
// WRONG: Wall of text without structure
The getUserById function is a utility that retrieves users from the
database and it's important because authentication depends on it and
you should be aware that it caches results for performance reasons...

// WRONG: Generic advice
This follows common patterns for database access.

// WRONG: No actionable context
**What it does**: Gets a user by ID
(So what? Why does this matter to the user?)

// WRONG: Assuming context
You probably know how JWT works, so I'll skip that part...
```

MUST generate:

```markdown
// RIGHT: Structured with clear sections
## getUserById

**What it does**: Fetches user record from PostgreSQL, returns null if not found.

**Key points**:
- Returns `null` on miss (does not throw)
- Auto-includes `profile` relation
- Cached 5 minutes in Redis

**Why it matters**: Safe to call without try/catch, but always check for null.

// RIGHT: Specific to their code
This follows the repository pattern used in `src/repositories/` —
see `orderRepository.ts:45` for a similar example.

// RIGHT: Actionable implication
**Why it matters**: The 5-minute cache means user updates won't reflect
immediately — call `invalidateUserCache(id)` after mutations.
```

## Validation

BEFORE returning explanation, verify:

- [ ] Directly answers user's question in first section
- [ ] Quick mode: readable in <30 seconds
- [ ] Deep mode: includes file:line references
- [ ] "Why it matters" contains actionable information
- [ ] No walls of unstructured text
- [ ] Offers exactly one follow-up option (not a list)
- [ ] If subject not found: alternatives offered

IF validation fails: Fix before returning.
