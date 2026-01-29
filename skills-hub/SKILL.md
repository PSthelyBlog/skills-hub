# Skills Hub

---
name: skills-hub
description: "Route requests to specialized skills, chain outputs for multi-step workflows"
version: 1.2.0
author: PSthelyBlog
tags: [orchestration, router, meta-skill, delegation]
model: sonnet
examples:
  - "/skills-hub explain the authentication flow"
  - "/skills-hub suggest improvements for src/api.ts"
  - "/skills-hub brainstorm database architecture then suggest implementation"
  - "/skills-hub update src/hooks/useAuth.ts"
  - "/skills-hub grounded what are the latest React 19 features"
  - "/skills-hub write-for-claude create a CLAUDE.md for this project"
---

## Trigger

WHEN user invokes `/skills-hub [request]`: Execute this protocol.

## Available Skills

| Skill | Purpose | Delegatable |
|-------|---------|-------------|
| explain | Understand code — quick answers to deep dives | Yes |
| suggest | Actionable code improvements with diffs | Yes |
| brainstorm | Multi-perspective debates (3 AI agents) | **No** — execute directly |
| update | Modernize code with fresh documentation | Yes |
| grounded | Verify volatile knowledge, trust stable knowledge | Yes |
| write-for-claude | Generate Claude-optimized instructions | Yes |

## Protocol

### Step 1: Parse Request

Extract from user input:

| Component | How to identify | Required |
|-----------|-----------------|----------|
| Explicit skill name | First word matches skill name | No |
| Chaining keywords | "then", "and then", "after that", "followed by" | No |
| Arguments | Everything after skill name(s) | Yes |

### Step 2: Route Request

#### 2.1 Routing Decision Tree

```
IF request starts with skill name (explain|suggest|brainstorm|update|grounded|write-for-claude):
  Route to that skill with remaining arguments
ELSE IF request contains chaining keyword:
  Parse into skill sequence (see Step 2.3)
ELSE:
  Apply intent-based routing (see Step 2.2)
```

#### 2.2 Intent-Based Routing

IF no explicit skill name, match intent pattern:

| IF request contains | THEN route to |
|---------------------|---------------|
| "what does", "how does", "why does", "explain" | explain |
| "improve", "better", "review", "suggest", "fix" | suggest |
| "should we", "trade-offs", "debate", "options", "decide" | brainstorm |
| "update", "modernize", "migrate", "upgrade", "latest" | update |
| "current", "latest version", "verify", "check if" | grounded |
| "write instructions", "create CLAUDE.md", "generate prompt", "SKILL.md" | write-for-claude |

IF multiple patterns match equally:
  Ask user: "Which skill should I use? [list matched skills with one-line descriptions]"

IF no pattern matches:
  Ask user: "Which skill should I use?" and list all available skills with descriptions.

#### 2.3 Parse Skill Chains

WHEN request contains chaining keyword ("then", "and then", "after that", "followed by"):

1. Split request at chaining keyword
2. Route each segment to appropriate skill
3. Return ordered sequence: `[skill-1, skill-2, ...]`

```
Example: "explain auth flow then suggest improvements"
→ Chain: [explain, suggest]
→ Arguments: ["auth flow", "improvements"]
```

Common valid chains:

| Chain | Use case |
|-------|----------|
| explain → suggest | Understand code, then improve it |
| explain → brainstorm | Understand system, then debate changes |
| brainstorm → suggest | Decide approach, then get implementation |
| grounded → update | Verify latest APIs, then modernize code |
| brainstorm → write-for-claude | Debate approach, then document decisions |

MUST NOT: Execute circular chains (skill appears twice)
  INSTEAD: Execute non-circular portion, ask user how to proceed

### Step 3: Execute Skills

#### 3.1 Execution Decision Tree

```
IF skill is brainstorm:
  Execute directly in main session (do not delegate)
  RATIONALE: brainstorm spawns 3 parallel Task agents; subagents cannot invoke Task
ELSE IF skills are independent (no data dependency between them):
  Launch all Task calls in parallel in single message
ELSE:
  Execute sequentially, passing output as context to next skill
```

#### 3.2 Single Skill Execution (Not Brainstorm)

MUST: Use Task tool with these parameters:

```
Task(
  subagent_type: "general-purpose",
  description: "Execute [skill-name] skill",
  prompt: """
    Execute the /[skill-name] skill with the following request:

    [user's arguments/query]

    Follow the skill protocol defined in ./skills-hub/[skill-name]/SKILL.md

    Return only the final output formatted according to the skill's output format.
  """
)
```

#### 3.3 Brainstorm Execution

WHEN routing to brainstorm:
  MUST NOT: Delegate via Task
  MUST: Execute brainstorm protocol directly in main session
  MUST: Read ./skills-hub/brainstorm/SKILL.md and follow its protocol

#### 3.4 Chained Execution

FOR each skill in chain (in order):

```
IF skill is brainstorm:
  Execute directly, store output
ELSE:
  Task(
    subagent_type: "general-purpose",
    description: "Execute [skill-name] with prior context",
    prompt: """
      Context from previous skill ([previous-skill-name]):
      ---
      [output from previous skill]
      ---

      Now execute the /[skill-name] skill:
      [user's query for this skill]

      Use the context above to inform your analysis.
      Follow the skill protocol in ./skills-hub/[skill-name]/SKILL.md
    """
  )
```

MUST: Pass previous skill output as context to next skill
MUST: Execute in order (no parallel execution for chains)

#### 3.5 Parallel Execution

WHEN skills are independent (different targets, no shared context):

MUST: Launch multiple Task calls in single message

```
Example: "/skills-hub explain src/api.ts and suggest improvements for src/utils.ts"

Launch in parallel:
- Task 1: explain src/api.ts
- Task 2: suggest src/utils.ts
```

### Step 4: Format Output

#### 4.1 Single Skill Result

```markdown
## [Skill Name] Result

[skill output verbatim]
```

#### 4.2 Chained Skill Results

```markdown
## Workflow: [skill-1] → [skill-2]

### [Skill-1]: [target/topic]

[skill-1 output]

---

### [Skill-2]: [target/topic]

[skill-2 output]

---

### Summary

[1-2 sentences synthesizing the combined result]
```

#### 4.3 Parallel Skill Results

```markdown
## Results

### [Skill-1]: [target]

[output]

### [Skill-2]: [target]

[output]
```

### Step 5: Handle Errors

WHEN skill execution fails:

```markdown
**[Skill Name] failed:** [error description]

Options:
1. Retry with modified parameters
2. Try a different skill
3. Proceed without this skill's output
```

WHEN skill name not recognized:

```markdown
Unknown skill: "[name]"

Available skills: explain, suggest, brainstorm, update, grounded, write-for-claude

Did you mean: [closest match based on string similarity]?
```

WHEN file argument does not exist:

```markdown
File not found: [path]

Similar files:
- [suggestion 1]
- [suggestion 2]
```

## Definitions

SKILL: A specialized protocol that Claude executes for a specific type of task
CHAIN: Ordered sequence of skills where each skill's output feeds into the next
DELEGATION: Executing a skill via Task subagent to preserve main session context
ROUTING: Matching user intent to the appropriate skill

## Edge Cases

WHEN request matches multiple skills equally:
  MUST: Ask user to choose
  MUST: Explain the difference between matched skills

WHEN chaining would create circular dependency:
  MUST NOT: Execute circular chains
  MUST: Execute non-circular portion
  MUST: Ask user how to proceed with remainder

WHEN skill requires file that does not exist:
  MUST: Return error with similar file suggestions (use Glob to find)

WHEN chained skill output format is incompatible with next skill:
  MUST: Transform output to appropriate format before passing

WHEN user asks for non-existent skill:
  MUST: Suggest closest available skill based on intent

WHEN brainstorm is in the middle of a chain (X → brainstorm → Y):
  MUST: Delegate X via Task
  MUST: Execute brainstorm directly with X output as context
  MUST: Delegate Y via Task with brainstorm output as context

DEFAULT: If uncertain about routing, ask user to specify skill.

## Anti-Patterns

MUST NOT: Delegate brainstorm skill

```markdown
// WRONG: Delegating brainstorm
Task(subagent_type: "general-purpose", prompt: "Execute /brainstorm...")

// RIGHT: Execute brainstorm directly
[Read brainstorm/SKILL.md and follow protocol in main session]
```

MUST NOT: Execute chains sequentially when skills are independent

```markdown
// WRONG: Sequential when parallel is possible
"/skills-hub explain src/a.ts and explain src/b.ts"
[Execute first Task, wait, execute second Task]

// RIGHT: Parallel execution
[Launch both Task calls in single message]
```

MUST NOT: Lose context between chained skills

```markdown
// WRONG: No context passing
Task(prompt: "Execute /suggest...")  // Missing explain output

// RIGHT: Pass previous output
Task(prompt: """
  Context from explain:
  ---
  [explain output]
  ---
  Now execute /suggest...
""")
```

MUST NOT: Guess skill when intent is ambiguous

```markdown
// WRONG: Assuming skill
User: "help with src/api.ts"
[Routes to explain without asking]

// RIGHT: Ask for clarification
"Which skill should I use?
- explain — understand what the code does
- suggest — get improvement recommendations"
```

## Skill Reference

| Skill | Triggers | Flags | Model |
|-------|----------|-------|-------|
| explain | "what", "how", "why", "explain" | `--deep` | haiku (sonnet with --deep) |
| suggest | "improve", "better", "review", "suggest" | `--category`, `--depth`, `--limit`, `--apply` | haiku |
| brainstorm | "should we", "trade-offs", "options", "decide" | `--agents`, `--transcript` | haiku |
| update | "update", "modernize", "migrate", "upgrade" | `--lib`, `--scope`, `--dry-run`, `--max-risk` | sonnet |
| grounded | "current", "latest", "verify" | none | sonnet |
| write-for-claude | "write instructions", "CLAUDE.md", "SKILL.md" | none | sonnet |

## Validation

BEFORE returning, verify:

- [ ] Correct skill(s) identified from request
- [ ] Brainstorm executed directly (not delegated) if present
- [ ] Independent skills launched in parallel
- [ ] Chained skills executed sequentially with context passing
- [ ] Output formatted according to section 4
- [ ] Errors handled with actionable options

IF validation fails: Fix before returning.
