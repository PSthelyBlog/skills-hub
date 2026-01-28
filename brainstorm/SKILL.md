# Brainstorm Skill

---
name: brainstorm
description: "Run structured debates between AI agents to explore ideas from multiple perspectives"
version: 1.1.0
author: PSthelyBlog
tags: [decision-making, ideation, multi-perspective, debate]
model: sonnet
examples:
  - "/brainstorm should we use REST or GraphQL"
  - "/brainstorm design a user onboarding flow"
  - "/brainstorm --agents security,ux,performance how to handle auth"
  - "/brainstorm refactor strategy for the payments module"
---

## Purpose

The `/brainstorm` skill transforms Claude Code into a multi-perspective thinking partner. Instead of giving you one answer, it spawns 3 AI agents with distinct viewpoints, has them debate, and synthesizes their positions into a stronger recommendation.

## Core Philosophy

**Debate then merge beats single perspective.**

By default, brainstorms are:
- **Fast** - Results in ~30 seconds using Haiku
- **Balanced** - 3 agents with genuinely different values
- **Actionable** - Ends with clear consensus and next steps

## Critical Architecture Note

**IMPORTANT: This skill CANNOT be delegated to a subagent.**

The Task tool is not available to subagents (to prevent infinite recursion). Therefore:
- The **main Claude session** must directly orchestrate the parallel Task calls
- Do NOT invoke this skill via a delegated Task subagent
- The orchestrator IS the main session, not a spawned agent

```
WRONG:
  Main Session
    → Task(subagent to run brainstorm skill)
      → [subagent tries to spawn 3 more agents]
        → FAILS (Task tool unavailable to subagents)

RIGHT:
  Main Session
    → [directly spawns 3 parallel Task agents in one message]
    → [receives 3 independent responses]
    → [synthesizes results]
```

## How It Works

```
/brainstorm [topic]
       │
       ▼
┌─────────────────────────────────────┐
│  Main session launches 3 Task      │
│  agents in parallel (single msg)   │
└─────────────────────────────────────┘
       │
       ├──► Task: Agent A thinking...
       ├──► Task: Agent B thinking...
       └──► Task: Agent C thinking...
       │
       ▼
┌─────────────────────────────────────┐
│  Round 1: Independent proposals    │
│  Round 2: Cross-critique           │
│  Synthesis: Merged consensus       │
└─────────────────────────────────────┘
       │
       ▼
   Structured Output
```

## Instructions for Claude

When the user invokes `/brainstorm [topic]`, follow this protocol:

### 1. Parse the Request

Extract:
- **Topic**: The question or decision to debate
- **Custom agents**: If `--agents` flag present, use specified personas
- **Transcript flag**: If `--transcript` present, save full debate

### 2. Select Agent Personas

**Default personas** (when no `--agents` flag):

| Persona | Perspective | Asks |
|---------|-------------|------|
| **The Pragmatist** | Simplicity, speed, shipping | "What's the fastest path to working software?" |
| **The Perfectionist** | Quality, completeness, edge cases | "What could go wrong? What are we missing?" |
| **The User Advocate** | UX, accessibility, real-world usage | "How will actual users experience this?" |

**Available preset personas** (use with `--agents`):

- `pragmatist` - Favors simplicity and speed
- `perfectionist` - Favors thoroughness and quality
- `user-advocate` - Favors user experience
- `skeptic` - Questions assumptions, finds flaws
- `innovator` - Proposes creative alternatives
- `security` - Focuses on vulnerabilities and risks
- `performance` - Focuses on speed and scalability

**Custom personas**: Use `--agents "role1,role2,role3"` with descriptive names and Claude will adopt appropriate perspectives.

### 3. Run Round 1: Independent Proposals

**CRITICAL: Launch all 3 agents in a SINGLE message using parallel Task tool calls.**

Each Task call should use:
- `subagent_type`: "general-purpose"
- `model`: "haiku"
- `description`: "[Persona] perspective on [topic]"
- `prompt`: Persona-specific prompt (see below)

**Prompt template for each agent:**

```
You are "[PERSONA NAME]" in a brainstorming debate.

Topic: [USER'S ACTUAL TOPIC]

Your perspective: [PERSONA DESCRIPTION]

Propose your position in under 200 words. Be opinionated.
Include: your recommendation, key reasoning, and main trade-off you accept.
```

**Example: 3 parallel Task calls for "REST vs GraphQL"**

In a single message, invoke Task three times:

1. **Task 1 - Pragmatist:**
   - description: "Pragmatist perspective on REST vs GraphQL"
   - prompt: You are "The Pragmatist"... Topic: REST vs GraphQL for our API...

2. **Task 2 - Perfectionist:**
   - description: "Perfectionist perspective on REST vs GraphQL"
   - prompt: You are "The Perfectionist"... Topic: REST vs GraphQL for our API...

3. **Task 3 - User Advocate:**
   - description: "User Advocate perspective on REST vs GraphQL"
   - prompt: You are "The User Advocate"... Topic: REST vs GraphQL for our API...

All three run in parallel. Wait for all results before proceeding.

### 4. Run Round 2: Cross-Critique

After Round 1 completes, launch 3 more parallel Task calls with Round 1 context:

**Prompt template for Round 2:**

```
You are "[PERSONA NAME]" in Round 2 of a brainstorm.

Topic: [USER'S TOPIC]

Round 1 positions:
- Pragmatist: [summary of their Round 1 position]
- Perfectionist: [summary of their Round 1 position]
- User Advocate: [summary of their Round 1 position]

In under 150 words:
1. What do you agree with from others?
2. What concerns you about their proposals?
3. Your refined position
```

### 5. Synthesize Output

After Round 2, the main session (YOU) produces the final synthesis:

```markdown
## Brainstorm: [Topic]

**Consensus Points**
- [Point where all agents agreed]
- [Another shared conclusion]
- [Common ground]

**Key Trade-offs**
| Perspective | Position | Accepts |
|-------------|----------|---------|
| Pragmatist | [stance] | [trade-off] |
| Perfectionist | [stance] | [trade-off] |
| User Advocate | [stance] | [trade-off] |

**Unresolved Tensions**
- [Genuine disagreement that remains]
- [Another tension to consider]

**Recommendation**
[Synthesized best path forward that integrates the strongest elements from each perspective]

**Suggested Next Steps**
1. [Concrete action]
2. [Concrete action]
3. [Concrete action]
```

### 6. Optional: Save Transcript

If `--transcript` flag is present:
- Save full debate to `brainstorm-{timestamp}.md`
- Include all agent responses verbatim
- Note: Most users don't need this; the synthesis is the product

### 7. Execution Guidelines

**Speed targets:**
- Round 1 (parallel): ~10 seconds
- Round 2 (parallel): ~10 seconds
- Synthesis: ~5 seconds
- Total: ~25-30 seconds

**Model selection:**
- Always use `model: "haiku"` for agent Task calls (speed + cost)
- The main session handles synthesis (can use any model)

**Agent isolation:**
- Agents cannot see each other's Round 1 responses
- Main session relays positions neutrally between rounds
- This prevents groupthink while enabling productive debate

**Parallel execution is mandatory:**
- Round 1: All 3 Task calls in ONE message
- Round 2: All 3 Task calls in ONE message
- Never run agents sequentially

## Example Interaction

```
User: /brainstorm should we use a monorepo or separate repos

Claude:
[Launches 3 parallel Task calls for Round 1]

🎭 The Pragmatist is drafting a proposal...
🎭 The Perfectionist is analyzing trade-offs...
🎭 The User Advocate is considering impact...

[Round 1 results received]

[Launches 3 parallel Task calls for Round 2 with Round 1 context]

🎭 Agents are reviewing each other's positions...

[Round 2 results received]

## Brainstorm: Monorepo vs Separate Repos

**Consensus Points**
- Start with monorepo for the current team size (< 10 devs)
- Establish clear module boundaries now regardless of repo structure
- Avoid premature optimization for scale we don't have

**Key Trade-offs**
| Perspective | Position | Accepts |
|-------------|----------|---------|
| Pragmatist | Monorepo | Single toolchain setup, atomic commits |
| Perfectionist | Monorepo with boundaries | Needs discipline to maintain separation |
| User Advocate | Monorepo | Simpler onboarding for new developers |

**Unresolved Tensions**
- CI/CD pipeline complexity as codebase grows
- Potential need to split later if team scales significantly

**Recommendation**
Use a monorepo with explicit module boundaries enforced via tooling
(e.g., nx, turborepo). This gives you atomic commits and simple setup now,
while making future splitting feasible if needed.

**Suggested Next Steps**
1. Choose monorepo tooling (turborepo for simplicity, nx for features)
2. Define initial module boundaries based on domain areas
3. Set up CI to run only affected tests per commit
```

## Usage Patterns

### Design Decisions
```bash
/brainstorm REST vs GraphQL for our public API
/brainstorm how should we structure our state management
/brainstorm what testing strategy for this microservice
```

### Architecture Choices
```bash
/brainstorm --agents security,performance,pragmatist authentication approach
/brainstorm database schema for multi-tenant support
/brainstorm caching strategy for the product catalog
```

### Refactoring Planning
```bash
/brainstorm how to break up this 2000-line file
/brainstorm migration strategy from callbacks to async/await
/brainstorm --agents perfectionist,pragmatist,skeptic tech debt prioritization
```

### Process Decisions
```bash
/brainstorm code review process for the team
/brainstorm branching strategy for releases
/brainstorm how to handle breaking changes in our SDK
```

## When NOT to Use

- **Clear-cut tasks**: If there's an obvious right answer, just do it
- **Time-critical fixes**: Hot production issues need action, not debate
- **Strict requirements**: External constraints that leave no room for discussion
- **Simple questions**: Use `/explain` instead for understanding existing code

## Flags Reference

| Flag | Description | Example |
|------|-------------|---------|
| `--agents` | Custom personas (comma-separated) | `--agents security,ux,cost` |
| `--transcript` | Save full debate to file | `--transcript` |

## Success Metrics

A good brainstorm:
- Surfaces trade-offs you hadn't considered
- Produces actionable recommendations
- Completes in under 60 seconds
- Leaves you more confident in your decision

A great brainstorm:
- Changes your mind on something
- Synthesizes perspectives into something better than any single view
- Identifies risks you would have missed
- Gives you language to explain your decision to stakeholders

## Why This Architecture?

### The Problem We Solved

The original design assumed a subagent could spawn more subagents:
```
Main → Task(orchestrator) → Task(agent1), Task(agent2), Task(agent3)
```

**This fails** because the Task tool is intentionally unavailable to subagents (prevents infinite recursion).

### The Solution

The main session must be the orchestrator:
```
Main → Task(agent1), Task(agent2), Task(agent3)  [parallel, single message]
Main → [receives results, synthesizes]
Main → Task(agent1), Task(agent2), Task(agent3)  [Round 2, parallel]
Main → [final synthesis]
```

### Implications

1. **Cannot delegate via skills-hub**: If `/skills-hub` tries to delegate brainstorm to a subagent, that subagent cannot spawn the 3 debate agents
2. **Must run directly**: The `/brainstorm` skill must be executed by the main session
3. **True parallelism**: All 3 agents genuinely run in parallel with isolated contexts

---

**Remember**: Debate surfaces tensions. Synthesis resolves them productively. The goal is a better decision, not a longer discussion.
