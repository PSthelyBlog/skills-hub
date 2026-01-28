# Design Rationale: The Brainstorm Skill v2.0

## Critical Discovery: Subagent Limitation

### The Original Assumption (v1.0)

The original design assumed this execution model:
```
User invokes /brainstorm
  → Skills-hub delegates to Task(orchestrator subagent)
    → Orchestrator spawns Task(agent1), Task(agent2), Task(agent3)
      → Agents debate, orchestrator synthesizes
```

### What We Discovered

**The Task tool is NOT available to subagents.**

When we tested this empirically, the subagent reported:

> "TASK TOOL NOT AVAILABLE. The Task tool for spawning parallel subagents
> is not available in my current tool set."

Subagents have access to: Bash, Glob, Grep, Read, Edit, Write, WebFetch, WebSearch, and MCP tools. But **not** the Task tool.

This is a deliberate design decision to prevent:
- Infinite recursion (Agent A spawns Agent B spawns Agent A...)
- Context explosion
- Runaway resource consumption

### The v1.0 Failure Mode

When v1.0 was invoked via skills-hub delegation:
1. Skills-hub spawned a subagent to "run the brainstorm skill"
2. The subagent read SKILL.md instructions to spawn 3 agents
3. The subagent had no Task tool available
4. Instead of failing explicitly, it improvised a **theatrical debate**
5. One model wrote a script pretending to be 3 agents arguing

This produced plausible-looking output, but:
- No actual parallel execution
- No true context isolation
- Single perspective dressed up as multiple
- The "debate" was one model's imagination

### The v2.0 Solution

```
User invokes /brainstorm
  → Main session (NOT a subagent) directly orchestrates
    → Main session spawns Task(agent1), Task(agent2), Task(agent3) in parallel
    → Three separate contexts, three genuine perspectives
    → Main session receives results and synthesizes
```

**Key insight**: The orchestrator MUST be the main session, not a delegated subagent.

---

## Synthesis Overview

This skill emerged from a three-agent brainstorm that debated how to design `/brainstorm` itself:

| Agent | Position | Core Value |
|-------|----------|------------|
| **Pragmatist** | Minimal config, hard-coded personas, 1 round | Speed, simplicity, "just works" |
| **Completionist** | Rich CLI, 15+ presets, custom agent files, voting | Full configurability, power users |
| **User Advocate** | Real-time progress, relatable personas, onboarding | Transparency, learning, accessibility |

## Key Design Decisions

### 1. Main Session as Orchestrator (NEW in v2.0)

**Problem**: How to achieve true multi-agent execution?

**v1.0 approach**: Delegate to orchestrator subagent (FAILED)
**v2.0 approach**: Main session directly orchestrates

**Why**:
- Task tool only available at main session level
- Guarantees true parallel execution
- No theatrical fallback possible
- Explicit failure if something goes wrong

**Trade-off accepted**: Cannot be cleanly delegated via skills-hub. The main session must handle brainstorm directly.

### 2. Two Rounds, Not One or Three

**Problem**: How many rounds of debate?

**Pragmatist wanted 1**: "Faster, simpler"
**Completionist wanted 3+**: "More thorough convergence"

**Solution**: 2 rounds (proposal + critique)

**Why**:
- Round 1: Independent proposals (no groupthink)
- Round 2: Cross-critique (surface tensions)
- More rounds have diminishing returns
- 2 rounds fits in ~30 seconds with Haiku

### 3. Mandatory Parallel Execution

**Problem**: Sequential vs parallel agent execution?

**Solution**: All agents in each round MUST be launched in a single message.

**Why**:
- True parallelism (3 agents run simultaneously)
- Faster execution (~10 seconds per round, not ~30)
- Guarantees context isolation (agents can't see each other's responses)
- The only way to achieve genuine multi-perspective debate

**Implementation**:
```
Round 1: Single message with 3 Task tool invocations
[Wait for all 3 results]
Round 2: Single message with 3 Task tool invocations
[Wait for all 3 results]
Synthesize
```

### 4. Haiku Model for Agents

**Problem**: Which model for agent tasks?

**Pragmatist**: "Haiku, always. Speed matters."
**Completionist**: "Sonnet for depth. Per-agent model selection."

**Solution**: Haiku for all agents, always.

**Why**:
- 6 Haiku calls << 6 Sonnet calls (cost)
- Parallel execution negates speed difference per-agent
- Agent responses are short (200 words) - Haiku handles well
- Main session (any model) handles synthesis

### 5. 7 Preset Personas

**Pragmatist wanted 3**: "Hard-code Pragmatist, Completionist, User Advocate"
**Completionist wanted 15+**: "Cover every domain imaginable"

**Solution**: 7 presets with custom `--agents` escape hatch

| Preset | Perspective |
|--------|-------------|
| pragmatist | Speed, simplicity |
| perfectionist | Quality, edge cases |
| user-advocate | UX, accessibility |
| skeptic | Questions assumptions |
| innovator | Creative alternatives |
| security | Vulnerabilities |
| performance | Scalability |

**Why**:
- 7 covers most scenarios without overwhelming
- Custom `--agents` handles edge cases
- Prevents endless "add persona X" requests

### 6. Single Flag: `--agents`

**Completionist wanted**: `--rounds`, `--model`, `--temperature`, `--config`
**Pragmatist wanted**: Zero flags

**Solution**: One flag: `--agents`

**Why**:
- Covers 90% of customization needs
- No cognitive load choosing options
- `--transcript` as secondary flag for debugging

---

## What We Rejected

### Rejected: Delegated Orchestration

**Why not**: Task tool unavailable to subagents. Doesn't work.

### Rejected: Theatrical Debate (Single Model)

**Why not**: One model pretending to be three isn't true multi-perspective. It's creative writing, not synthesis.

### Rejected: Sequential Agent Execution

**Why not**: Slower (3x time), and agents could potentially see previous outputs (no true isolation).

### Rejected: Configurable Round Count

**Why not**: Diminishing returns after 2 rounds. Adds complexity without value.

### Rejected: Agent Memory Across Sessions

**Why not**: Each brainstorm should be fresh. Past positions could bias.

---

## Contribution From Each Agent

### Pragmatist Contributions
- Haiku model for all agents
- ~30 second total execution
- Zero config needed to start
- Single `--agents` flag

### Completionist Contributions
- 7 preset personas (not just 3)
- Custom `--agents` flag
- Structured 5-section output
- `--transcript` option

### User Advocate Contributions
- Clear progress indication
- Relatable persona names
- Actionable "Next Steps" section
- Unresolved tensions explicitly stated

---

## v2.0 Changes Summary

| Aspect | v1.0 | v2.0 |
|--------|------|------|
| Orchestrator | Delegated subagent | Main session directly |
| Agent spawning | Assumed nested Task | Confirmed: parallel Task from main |
| Failure mode | Theatrical fallback | Explicit failure |
| Skills-hub integration | Delegated | Cannot delegate; run directly |
| Architecture doc | Assumed working | Documents actual limitation |

---

## Lessons Learned

### 1. Test assumptions empirically
The v1.0 design assumed nested Task calls worked. A simple test revealed they don't.

### 2. Subagent tools are limited
Subagents have Read/Write/Bash/Search but NOT Task. This is intentional.

### 3. Main session orchestration is required for multi-agent
True multi-agent patterns require the main session to spawn agents directly.

### 4. Theatrical fallbacks mask failures
When the subagent couldn't spawn agents, it wrote a fake debate. This looked like success but wasn't.

---

## Open Questions

### Should skills-hub handle this specially?

Skills-hub could detect brainstorm invocations and NOT delegate them. Instead, it could instruct the main session to run brainstorm directly.

**Current status**: Documented as limitation. User should invoke `/brainstorm` directly.

### Could we add Task tool to subagents with limits?

Theoretically, Task could be available to subagents with:
- Depth limit (max 1 level of nesting)
- Count limit (max N spawned agents)
- No recursive self-spawning

**Current status**: Not implemented. Would require Claude Code changes.

---

## Conclusion

The v2.0 brainstorm skill acknowledges reality:
- **Task tool is main-session only**
- **True multi-agent requires direct orchestration**
- **Theatrical fallbacks are failures, not features**

The skill now documents this clearly and provides correct instructions for genuine multi-perspective brainstorming.
