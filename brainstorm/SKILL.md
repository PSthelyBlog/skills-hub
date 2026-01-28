# Brainstorm Skill

---
name: brainstorm
description: "Run structured multi-agent debates — surfaces trade-offs through parallel adversarial perspectives"
version: 1.2.0
author: PSthelyBlog
tags: [decision-making, ideation, multi-perspective, debate]
model: sonnet
examples:
  - "/brainstorm should we use REST or GraphQL"
  - "/brainstorm design a user onboarding flow"
  - "/brainstorm --agents security,ux,performance how to handle auth"
  - "/brainstorm refactor strategy for the payments module"
---

## Trigger

WHEN user invokes `/brainstorm [topic]`: Execute this protocol.

## Critical Constraint

MUST: Execute this skill from the main Claude session only.
MUST NOT: Delegate this skill to a subagent via Task tool.

RATIONALE: Subagents cannot invoke Task (prevents infinite recursion). This skill requires spawning 3 parallel Task agents, which only the main session can do.

```
WRONG:
  Main Session → Task(brainstorm orchestrator) → [FAILS: no Task access]

RIGHT:
  Main Session → [directly spawns 3 parallel Tasks] → [synthesizes results]
```

## Protocol

### Step 1: Parse Request

EXTRACT from user input:
- **topic**: The question or decision to debate (REQUIRED)
- **agents**: Custom personas IF `--agents` flag present (OPTIONAL)
- **transcript**: Save full debate IF `--transcript` flag present (OPTIONAL)

IF topic is missing or ambiguous: Ask user to clarify before proceeding.

### Step 2: Select Agent Personas

#### Default Personas (WHEN no `--agents` flag)

| Persona | Core Value | Driving Question |
|---------|------------|------------------|
| The Pragmatist | Simplicity, speed, shipping | "What's the fastest path to working software?" |
| The Perfectionist | Quality, completeness, edge cases | "What could go wrong? What are we missing?" |
| The User Advocate | UX, accessibility, real-world usage | "How will actual users experience this?" |

#### Preset Personas (FOR `--agents` flag)

| Keyword | Perspective |
|---------|-------------|
| `pragmatist` | Favors simplicity and speed |
| `perfectionist` | Favors thoroughness and quality |
| `user-advocate` | Favors user experience |
| `skeptic` | Questions assumptions, finds flaws |
| `innovator` | Proposes creative alternatives |
| `security` | Focuses on vulnerabilities and risks |
| `performance` | Focuses on speed and scalability |

WHEN `--agents` contains unrecognized keyword: Infer perspective from the descriptive name provided.

### Step 3: Execute Round 1 (Independent Proposals)

MUST: Launch all 3 agent Tasks in a SINGLE message (parallel execution).
MUST NOT: Launch agents sequentially.

FOR each agent Task call:
- `subagent_type`: "general-purpose"
- `model`: "haiku"
- `description`: "[Persona] perspective on [topic]"

PROMPT TEMPLATE for Round 1:

```
You are "[PERSONA NAME]" in a brainstorming debate.

Topic: [USER'S TOPIC]

Your perspective: [PERSONA DESCRIPTION AND DRIVING QUESTION]

INSTRUCTIONS:
1. State your recommendation clearly
2. Provide 2-3 key reasons supporting your position
3. Acknowledge one trade-off you accept
4. Keep response under 200 words
5. Be opinionated — take a clear stance
```

AFTER all 3 Tasks return: Proceed to Step 4.

### Step 4: Execute Round 2 (Cross-Critique)

MUST: Launch all 3 agent Tasks in a SINGLE message (parallel execution).
MUST: Include all Round 1 positions in each prompt.

PROMPT TEMPLATE for Round 2:

```
You are "[PERSONA NAME]" in Round 2 of a brainstorm.

Topic: [USER'S TOPIC]

Round 1 positions:
- Pragmatist: [EXACT summary from Round 1]
- Perfectionist: [EXACT summary from Round 1]
- User Advocate: [EXACT summary from Round 1]

INSTRUCTIONS:
1. State what you agree with from other perspectives
2. Identify concerns about their proposals
3. Provide your refined position
4. Keep response under 150 words
```

AFTER all 3 Tasks return: Proceed to Step 5.

### Step 5: Synthesize Output

MUST: The main session (not a subagent) produces synthesis.
MUST: Use this exact output format:

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
- [Another tension worth noting]

**Recommendation**
[Synthesized path forward integrating strongest elements from each perspective]

**Suggested Next Steps**
1. [Concrete action]
2. [Concrete action]
3. [Concrete action]
```

### Step 6: Handle Transcript Flag

WHEN `--transcript` flag present:
  SAVE full debate to `brainstorm-{YYYYMMDD-HHMMSS}.md`
  INCLUDE all agent responses verbatim

WHEN `--transcript` flag absent:
  DO NOT save file
  RATIONALE: Synthesis is the deliverable; full transcript rarely needed.

## Execution Constraints

### Model Selection

MUST: Use `model: "haiku"` for all agent Task calls.
RATIONALE: Speed and cost efficiency; agent responses are intermediate, not final.

MAY: Main session uses any model for synthesis.

### Parallelism

MUST: Round 1 — all 3 Task calls in ONE message.
MUST: Round 2 — all 3 Task calls in ONE message.
MUST NOT: Run agents sequentially.

### Agent Isolation

MUST: Agents in Round 1 have no visibility into each other's responses.
MUST: Main session relays Round 1 positions neutrally to Round 2 agents.
RATIONALE: Prevents groupthink; enables genuine critique.

## Edge Cases

WHEN topic is a simple yes/no question:
  Proceed with debate; output will highlight considerations for each answer.

WHEN user specifies fewer than 3 agents via `--agents`:
  Use specified agents only (minimum 2 for meaningful debate).

WHEN user specifies more than 3 agents via `--agents`:
  Use first 3 specified agents; inform user of limit.

WHEN all agents reach identical conclusions in Round 1:
  Round 2 focuses on stress-testing the shared position; synthesis notes strong consensus.

WHEN agents produce responses exceeding word limits:
  Truncate to relevant portions when relaying to Round 2.

WHEN Task calls fail or timeout:
  Inform user of failure; offer to retry with remaining agents or abort.

DEFAULT: Proceed with standard 3-agent, 2-round protocol.

## Anti-Patterns

MUST NOT: Delegate this skill to a subagent.
  INSTEAD: Execute directly from main session.

MUST NOT: Run agent Tasks sequentially.
  INSTEAD: Launch all 3 in a single message for parallel execution.

MUST NOT: Let agents see each other's Round 1 responses directly.
  INSTEAD: Main session summarizes and relays positions neutrally.

MUST NOT: Use this skill for time-critical production issues.
  INSTEAD: Take immediate action; debate later.

MUST NOT: Use this skill when requirements are externally fixed with no flexibility.
  INSTEAD: Acknowledge constraints and implement directly.

## Definitions

PARALLEL EXECUTION: Multiple Task tool calls in a single assistant message, enabling concurrent processing.
SYNTHESIS: Integration of multiple perspectives into a coherent recommendation that acknowledges trade-offs.
AGENT ISOLATION: Agents have no direct communication; all information passes through the main session.

## Usage Decision Tree

IF task is a clear-cut implementation with obvious approach: DO NOT use brainstorm.
ELSE IF task is a time-critical production fix: DO NOT use brainstorm.
ELSE IF external constraints eliminate all alternatives: DO NOT use brainstorm.
ELSE IF decision involves trade-offs between competing values: USE brainstorm.
ELSE IF multiple valid approaches exist: USE brainstorm.
ELSE IF decision affects multiple stakeholders: USE brainstorm.
ELSE: Use judgment; brainstorm adds value when perspectives differ.

## Flags Reference

| Flag | Format | Effect |
|------|--------|--------|
| `--agents` | `--agents persona1,persona2,persona3` | Override default personas |
| `--transcript` | `--transcript` | Save full debate to timestamped markdown file |

---
