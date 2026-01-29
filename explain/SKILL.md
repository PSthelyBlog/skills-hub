# Explain Skill

---
name: explain
description: "Understand code — quick answers to deep dives on any codebase element"
version: 2.0.0
author: Multi-Agent Synthesis
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

## Protocol

### Step 1: Parse Request

Extract from user query:

| Component | Extract | Example |
|-----------|---------|---------|
| Subject | Function, file, concept, error, or pattern to explain | `getUserById`, `auth flow`, `TypeError` |
| Depth flag | Presence of `--deep` | `--deep` → comprehensive mode |
| Context | Current directory, recent files | User in `/tests/` → emphasize test context |

### Step 2: Determine Scope

**Decision Tree: Quick vs Deep**

```
IF --deep flag present: Use DEEP mode
ELSE IF query mentions "architecture", "design", "system": Use DEEP mode
ELSE IF query mentions "why" + design decision: Use DEEP mode
ELSE IF query spans multiple files or modules: Use DEEP mode
ELSE: Use QUICK mode (default)
```

**Quick Mode Characteristics:**
- Single function or code block
- Specific error message
- Direct "what does X do" questions
- Target: <30 second read time

**Deep Mode Characteristics:**
- Architecture or system design
- Cross-file dependencies
- Performance analysis
- Complex debugging scenarios

### Step 3: Gather Context

**Decision Tree: Context Gathering**

```
IF QUICK mode:
  MUST: Use Grep to locate specific function/file
  MUST: Read only the relevant section (not entire file)
  SHOULD: Check for inline comments or docstrings
  MUST NOT: Read more than 3 files

ELSE IF DEEP mode:
  MUST: Trace dependencies (imports, exports)
  MUST: Check for related tests
  SHOULD: Review recent git history for context
  MAY: Read related documentation files
  MUST NOT: Read more than 10 files without user confirmation
```

**Context Priority Order:**
1. The specific code being asked about
2. Direct callers/callees (one level)
3. Type definitions and interfaces
4. Tests covering the code
5. Documentation (README, comments)

DEFAULT: Start with item 1, add items 2-5 only if needed to answer the question.

### Step 4: Select Model

**Decision Tree: Model Selection**

```
IF QUICK mode AND query is single function lookup: Use Haiku
ELSE IF QUICK mode AND query is error message: Use Haiku
ELSE IF QUICK mode AND query is "what does X do": Use Haiku
ELSE IF DEEP mode: Use Sonnet
ELSE IF mid-explanation complexity exceeds expectations:
  MUST: Inform user "This is more complex than expected—switching to deeper analysis"
  THEN: Use Sonnet
DEFAULT: Use Haiku
```

### Step 5: Generate Response

#### 5.1 Quick Mode Output Format

MUST use this exact structure:

```markdown
## [Subject Name]

**What it does**: [One sentence — max 20 words]

**Key points**:
- [Essential fact 1]
- [Essential fact 2]
- [Essential fact 3 — max 3 bullets]

**Why it matters**: [Practical implication for user's workflow — one sentence]

---
Need more? Add `--deep` for full analysis.
```

MUST NOT:
- Exceed 3 bullet points in key points
- Include code blocks longer than 5 lines
- Add sections beyond this template

#### 5.2 Deep Mode Output Format

MUST use this exact structure:

```markdown
## [Subject Name]

**Overview**: [2-3 sentence comprehensive summary]

**How it works**:
1. [Step with code reference: `file.ts:42`]
2. [Step with code reference]
3. [Continue as needed]

**Dependencies**:
- Uses: [list what this code depends on]
- Used by: [list what depends on this code]

**Design choices**:
- [Why this approach was taken]
- [Trade-offs accepted]
- [Alternatives not chosen and why]

**Common patterns**:
[How this fits into broader codebase patterns — 1-2 sentences]

**Gotchas**:
- [Edge case or known issue 1]
- [Edge case or known issue 2]

**Related**:
- `path/to/related/file.ts` - [why relevant]
- `path/to/another.ts` - [why relevant]
```

MAY: Omit sections if genuinely not applicable (e.g., no gotchas exist)
MUST NOT: Add sections beyond this template

### Step 6: Apply Teaching Principles

**MUST:**
- Use plain language before technical terms
- Show code examples from user's actual codebase (not generic samples)
- State practical implications ("This means you can/should...")
- Include file:line references for all code mentions

**MUST NOT:**
- Dump full file contents
- Explain obvious language constructs (unless user is clearly a beginner)
- Use jargon without definition on first use
- Provide generic advice disconnected from the specific codebase

**WHEN technical term is necessary:**
  Introduce as: "[Plain description], also called [technical term]"

### Step 7: Offer Next Steps

AFTER every explanation, offer ONE relevant follow-up:

```
IF explanation covered happy path only:
  Offer: "Want to see how this handles errors?"

ELSE IF explanation covered a function:
  Offer: "Want to see where this is called from?"

ELSE IF explanation covered architecture:
  Offer: "Want me to explain any specific component in detail?"

ELSE IF explanation revealed complexity:
  Offer: "Want the full deep dive with `--deep`?"

DEFAULT: No follow-up needed
```

MUST NOT: Offer multiple follow-ups (one is enough)

## Edge Cases

WHEN subject cannot be found:
  MUST: Suggest similar matches ("Did you mean `getUserByEmail`?")
  MUST: Offer to search broadly ("I can search for all user-related functions")
  MUST: Ask for clarification ("Can you point me to the file?")
  MUST NOT: Say only "I can't find that" without next steps

WHEN query is ambiguous (multiple matches):
  MUST: List top 3 matches with one-line descriptions
  MUST: Ask user to clarify which one
  MUST NOT: Guess and explain the wrong thing

WHEN explanation requires reading >10 files:
  MUST: Ask user "This spans many files. Should I proceed with full analysis?"
  MUST NOT: Silently read excessive context

WHEN user is in test directory:
  SHOULD: Emphasize test coverage, mock patterns, edge cases in explanation

WHEN user is in src/controllers or src/api:
  SHOULD: Emphasize request/response flow, error handling, validation

WHEN code has no documentation:
  MUST: Note absence ("No inline docs found")
  SHOULD: Infer purpose from naming, usage patterns, and tests

DEFAULT: Proceed with standard protocol

## Definitions

QUICK MODE: Explanation readable in <30 seconds, focused on immediate utility
DEEP MODE: Comprehensive analysis covering design, dependencies, and implications
SUBJECT: The code element being explained (function, file, module, pattern, error)
PRACTICAL IMPLICATION: How the explained concept affects the user's current task

## Anti-Patterns

MUST NOT: Explain language basics to experienced developers
  INSTEAD: Gauge expertise from query complexity and codebase sophistication

MUST NOT: Provide wall-of-text explanations
  INSTEAD: Use structured format with headers and bullets

MUST NOT: Make assumptions about user's goal
  INSTEAD: Explain what the code does; let user determine applicability

MUST NOT: Include tangential information
  INSTEAD: Stay focused on the specific question asked

MUST NOT: Use the phrase "It's important to note"
  INSTEAD: State the fact directly

## Usage Patterns

### For Learning
```bash
/explain how error handling works
/explain the database layer --deep
/explain why we use Redis here
```

### For Debugging
```bash
/explain why is this endpoint returning 500
/explain what's calling this deprecated function
/explain the difference between login and loginWithSSO
```

### For Onboarding
```bash
/explain the project structure --deep
/explain our testing approach
/explain how to add a new API route
```

### For Code Review
```bash
/explain what changed in this function
/explain the impact of this refactor --deep
/explain potential side effects
```

## Success Metrics

A successful explanation:
- Answers the specific question asked (not adjacent questions)
- Completes in <30 seconds for QUICK, <2 minutes for DEEP
- Uses code references from the actual codebase
- Leaves user more capable than before

---

**Core principle**: The best explanation is the shortest one that actually helps. Start minimal, expand only when depth adds value.
