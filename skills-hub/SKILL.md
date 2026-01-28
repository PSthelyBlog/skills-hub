# Skills Hub

---
name: skills-hub
description: "Orchestrate specialized Claude Code skills — route requests to the right skill and combine outputs intelligently"
version: 1.1.0
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

## Purpose

The `/skills-hub` command is the unified entry point for all available Claude Code skills. It analyzes user requests, routes them to the appropriate skill(s), and can chain multiple skills together for complex workflows.

## Core Philosophy

**Delegation preserves context, composition multiplies power.**

The orchestrator:
- **Routes intelligently** - Matches user intent to the right skill
- **Delegates cleanly** - Uses Task subagents to execute skills
- **Chains seamlessly** - Combines skill outputs for multi-step workflows
- **Manages tokens** - Subagent execution prevents context contamination

## Available Skills

| Skill | Purpose | Use When |
|-------|---------|----------|
| **explain** | Understand code — quick answers to deep dives | "What does this do?", "How does X work?" |
| **suggest** | Actionable code improvements with diffs | "How can this be better?", "Review this code" |
| **brainstorm** ⚠️ | Multi-perspective debates (3 AI agents) — *cannot be delegated* | Design decisions, architecture choices, trade-offs |
| **update** | Modernize code with fresh documentation | Outdated dependencies, deprecated APIs, migrations |
| **grounded** | Verify volatile knowledge, trust stable knowledge | Current events, new APIs, regulations, time-sensitive info |
| **write-for-claude** | Generate Claude-optimized instructions | CLAUDE.md, SKILL.md, AGENTS.md, prompts |

## Instructions for Claude

WHEN user invokes `/skills-hub [request]`: Execute this protocol.

### Step 1: Parse the Request

Extract:
- **Explicit skill name**: IF request starts with skill name (explain, suggest, brainstorm, update, grounded, write-for-claude)
- **Chaining intent**: IF request contains "then", "and then", "after that", "followed by"
- **Arguments**: Everything after the skill name(s)

### Step 2: Route the Request

#### 2.1 Explicit Skill Invocation

IF request explicitly names a skill:

```
"/skills-hub explain src/auth.ts" → Route to explain skill
"/skills-hub suggest --category security" → Route to suggest skill
"/skills-hub brainstorm REST vs GraphQL" → Route to brainstorm skill
```

MUST: Pass all arguments after the skill name to the target skill.

#### 2.2 Intent-Based Routing

IF no explicit skill name, analyze intent:

| Intent Pattern | Route To |
|----------------|----------|
| "what does", "how does", "why does", "explain" | explain |
| "improve", "better", "review", "suggest", "fix" | suggest |
| "should we", "trade-offs", "debate", "options", "decide" | brainstorm |
| "update", "modernize", "migrate", "upgrade", "latest" | update |
| "current", "latest version", "verify", "check if" | grounded |
| "write instructions", "create CLAUDE.md", "generate prompt" | write-for-claude |

IF intent unclear:
```
Ask user:
"Which skill should I use?
- explain — understand what code does
- suggest — get improvement recommendations
- brainstorm — explore options from multiple perspectives
- update — modernize with current documentation
- grounded — verify time-sensitive information
- write-for-claude — generate Claude-optimized instructions"
```

#### 2.3 Skill Chaining

IF request contains chaining keywords ("then", "and then", "after that"):

Parse into ordered skill sequence:
```
"/skills-hub explain auth flow then suggest improvements"
→ Chain: [explain, suggest]
→ Pass explain output as context to suggest
```

Common chains:
| Chain | Use Case |
|-------|----------|
| explain → suggest | Understand code, then improve it |
| explain → brainstorm | Understand system, then debate changes |
| brainstorm → suggest | Decide approach, then get implementation suggestions |
| grounded → update | Verify latest APIs, then modernize code |
| brainstorm → write-for-claude | Debate approach, then document decisions |

**Note on brainstorm in chains:** Since brainstorm cannot be delegated, chains involving brainstorm require special handling:
- `X → brainstorm`: Delegate X via Task, then execute brainstorm directly with the result as context
- `brainstorm → Y`: Execute brainstorm directly, then delegate Y via Task with brainstorm output as context

### Step 3: Execute via Task Subagents

MUST: Use Task tool with subagent_type="general-purpose" for each skill.

**EXCEPTION: brainstorm skill cannot be delegated.**
The brainstorm skill spawns 3 parallel Task agents internally. Since subagents cannot invoke the Task tool (to prevent infinite recursion), brainstorm must be executed directly by the main session, not delegated to a subagent. When a request routes to brainstorm, the main session must follow the brainstorm skill protocol directly instead of wrapping it in a Task call.

**Why subagents (for other skills):**
- Prevents context contamination in main session
- Each skill runs with clean context
- Token-efficient — only results return to main context
- Skills can be run in parallel when independent

#### 3.1 Single Skill Execution

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

#### 3.2 Chained Skill Execution

FOR sequential chains:

```
Step 1: Execute first skill via Task
Step 2: Receive output
Step 3: Execute second skill via Task, passing previous output as context:

Task(
  subagent_type: "general-purpose",
  description: "Execute [skill-2] with [skill-1] context",
  prompt: """
    Context from previous skill ([skill-1]):
    ---
    [output from skill-1]
    ---

    Now execute the /[skill-2] skill:
    [user's query for skill-2]

    Use the context above to inform your analysis.
    Follow the skill protocol in ./skills-hub/[skill-2]/SKILL.md
  """
)
```

#### 3.3 Parallel Skill Execution

IF skills are independent (no data dependency):

MUST: Launch multiple Task calls in parallel in a single message.

```
Example: "/skills-hub explain src/api.ts and suggest improvements for src/utils.ts"

These are independent — launch both Tasks in one message:
- Task 1: explain src/api.ts
- Task 2: suggest src/utils.ts

Both run concurrently.
```

### Step 4: Synthesize Results

#### 4.1 Single Skill Result

Return the skill output directly with minimal wrapper:

```markdown
## [Skill Name] Result

[skill output]
```

#### 4.2 Chained Skill Results

Combine outputs with clear separation:

```markdown
## Workflow: [skill-1] → [skill-2]

### [Skill-1]: [brief description]

[skill-1 output]

---

### [Skill-2]: [brief description]

[skill-2 output]

---

### Summary

[1-2 sentence synthesis of the combined workflow result]
```

#### 4.3 Parallel Skill Results

Present side by side or sequentially:

```markdown
## Results

### [Skill-1]: [target]

[output]

### [Skill-2]: [target]

[output]
```

### Step 5: Handle Errors

WHEN skill execution fails:
```
⚠️ [Skill Name] encountered an issue:
[Error description]

Options:
1. Retry with modified parameters
2. Try a different skill
3. Proceed without this skill's output
```

WHEN skill not found:
```
Unknown skill: "[name]"

Available skills:
- explain, suggest, brainstorm, update, grounded, write-for-claude

Did you mean: [closest match]?
```

## Skill Reference

### explain
**Purpose:** Understand code — quick answers or deep analysis
**Triggers:** "what", "how", "why", "explain"
**Flags:** `--deep` for comprehensive analysis
**Model:** haiku (default), sonnet (--deep)

### suggest
**Purpose:** Actionable code improvements with before/after diffs
**Triggers:** "improve", "better", "review", "suggest"
**Flags:** `--category`, `--depth`, `--limit`, `--apply`
**Model:** haiku

### brainstorm
**Purpose:** Multi-perspective debates (3 AI personas)
**Triggers:** "should we", "trade-offs", "options", "decide"
**Flags:** `--agents`, `--transcript`
**Model:** haiku
**⚠️ Cannot be delegated:** Must be executed directly by the main session (spawns its own Task agents)

### update
**Purpose:** Modernize code using fresh Context7 documentation
**Triggers:** "update", "modernize", "migrate", "upgrade"
**Flags:** `--lib`, `--scope`, `--dry-run`, `--max-risk`
**Model:** sonnet

### grounded
**Purpose:** Epistemic calibration — verify volatile, trust stable
**Triggers:** "current", "latest", "verify", time-sensitive queries
**Model:** sonnet

### write-for-claude
**Purpose:** Generate Claude-optimized instructions
**Triggers:** "write instructions", "CLAUDE.md", "SKILL.md", "prompt"
**Output:** CLAUDE.md, SKILL.md, AGENTS.md, or prompt documents
**Model:** sonnet

## Common Workflows

### Code Understanding → Improvement
```bash
/skills-hub explain src/api/users.ts then suggest
```

### Decision Making → Implementation
```bash
/skills-hub brainstorm authentication approach then suggest implementation
```

### Learn → Decide → Document
```bash
/skills-hub explain the auth system
/skills-hub brainstorm improvements
/skills-hub write-for-claude document the new auth approach
```

### Modernization Workflow
```bash
/skills-hub grounded check React Query latest patterns
/skills-hub update src/hooks/useData.ts --lib @tanstack/react-query
```

## Edge Cases

WHEN request matches multiple skills equally:
  Ask user to clarify which skill to use, explaining the difference.

WHEN chaining would create circular dependency:
  MUST NOT: Execute circular chains (e.g., suggest → explain → suggest)
  INSTEAD: Execute the non-circular portion and ask user how to proceed.

WHEN skill requires file that doesn't exist:
  Return error with suggestions for similar files.

WHEN chained skill needs output format incompatible with next skill:
  Transform output to appropriate format before passing to next skill.

WHEN user asks for skill that doesn't exist:
  Suggest the closest available skill based on intent.

## Success Metrics

A good skills hub invocation:
- Routes to the correct skill(s) without ambiguity
- Executes skills efficiently (parallel when possible)
- Returns synthesized output that's immediately useful
- Maintains context between chained skills

A great skills hub invocation:
- Anticipates which skill chain would best serve the user's actual goal
- Suggests skill combinations the user didn't think to ask for
- Produces output that's better than any single skill alone

---

**Remember:** The hub is invisible infrastructure. Users should feel like they're working with one intelligent system, not managing multiple tools. Route fast, execute clean, synthesize smart.
