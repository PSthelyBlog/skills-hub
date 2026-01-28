# Brainstorm Skill for Claude Code (v2.0)

**Debate ideas from multiple perspectives - get stronger recommendations.**

## What It Does

The `/brainstorm` skill turns Claude Code into a multi-perspective thinking partner. Instead of getting one answer, it spawns 3 AI agents with distinct viewpoints, has them debate, and synthesizes their positions into a stronger recommendation.

## Important: Direct Invocation Required

**This skill must be invoked directly, NOT via skills-hub delegation.**

```bash
# CORRECT - invoke directly
/brainstorm should we use REST or GraphQL

# INCORRECT - delegation fails silently
/skills-hub brainstorm should we use REST or GraphQL
```

Why? The Task tool (needed to spawn debate agents) is not available to subagents. The main Claude session must orchestrate directly. See DESIGN.md for details.

## Quick Start

```bash
# Simple question
/brainstorm should we use REST or GraphQL

# Custom perspectives
/brainstorm --agents security,ux,performance how to handle auth

# Design decisions
/brainstorm refactor strategy for the payments module
```

## How It Works

1. **Round 1**: 3 agents independently propose positions (parallel execution)
2. **Round 2**: Agents review each other's positions and refine (parallel execution)
3. **Synthesis**: Main session merges perspectives into recommendation

All agents run in **parallel with isolated contexts** - they genuinely can't see each other's responses until the cross-critique round.

## Core Features

### True Multi-Agent Execution
- 3 separate Task agents with isolated contexts
- Parallel execution (not sequential, not theatrical)
- Genuine debate, not one model role-playing

### Real-Time Visibility
- Live progress indicators during execution
- See when each agent is thinking
- ~30 second total execution time

### Structured Synthesis
- Consensus points highlighted
- Trade-offs made explicit
- Unresolved tensions acknowledged
- Concrete next steps provided

## Example Session

```
You: /brainstorm monorepo vs separate repos

Claude:
[Spawns 3 parallel Task agents]

🎭 The Pragmatist is drafting a proposal...
🎭 The Perfectionist is analyzing trade-offs...
🎭 The User Advocate is considering impact...

[Round 1 completes - 3 independent results]

[Spawns 3 parallel Task agents with Round 1 context]

🎭 Agents are reviewing each other's positions...

[Round 2 completes]

## Brainstorm: Monorepo vs Separate Repos

**Consensus Points**
- Start with monorepo for the current team size (< 10 devs)
- Establish clear module boundaries now
- Avoid premature optimization for scale we don't have

**Key Trade-offs**
| Perspective | Position | Accepts |
|-------------|----------|---------|
| Pragmatist | Monorepo | Single toolchain, atomic commits |
| Perfectionist | Monorepo with boundaries | Discipline required |
| User Advocate | Monorepo | Simpler onboarding |

**Unresolved Tensions**
- CI/CD pipeline complexity as codebase grows
- Potential need to split later if team scales

**Recommendation**
Use a monorepo with explicit module boundaries enforced via tooling
(turborepo). Atomic commits now, easy to split later if needed.

**Suggested Next Steps**
1. Choose monorepo tooling (turborepo vs nx)
2. Define initial module boundaries
3. Set up CI to run only affected tests
```

## Available Personas

| Persona | Perspective | Asks |
|---------|-------------|------|
| `pragmatist` | Simplicity, speed | "What's the fastest path?" |
| `perfectionist` | Quality, edge cases | "What could go wrong?" |
| `user-advocate` | UX, accessibility | "How will users experience this?" |
| `skeptic` | Questions assumptions | "Are we solving the right problem?" |
| `innovator` | Creative alternatives | "Is there a better way?" |
| `security` | Vulnerabilities, risks | "How could this be exploited?" |
| `performance` | Speed, scalability | "Will this work at 10x scale?" |

## Flags

| Flag | Description | Example |
|------|-------------|---------|
| `--agents` | Custom personas | `--agents security,ux,cost` |
| `--transcript` | Save full debate | `--transcript` |

## Use Cases

### Design Decisions
```bash
/brainstorm REST vs GraphQL for our public API
/brainstorm how should we structure state management
```

### Architecture Choices
```bash
/brainstorm --agents security,performance,pragmatist auth approach
/brainstorm database schema for multi-tenant support
```

### Refactoring Strategy
```bash
/brainstorm how to break up this 2000-line file
/brainstorm migration from callbacks to async/await
```

## When NOT to Use

- **Clear-cut tasks**: Obvious answer? Just do it.
- **Time-critical fixes**: Production down? Act, don't debate.
- **Simple questions**: Use `/explain` for understanding code.

## Performance

- **Execution time**: ~30 seconds (parallel Haiku agents)
- **Rounds**: 2 (proposals + cross-critique)
- **Cost**: 6 Haiku calls total

## Troubleshooting

### "I see a theatrical dialogue instead of synthesis"

If output looks like a script with agents talking to each other, something went wrong. This means one model is role-playing, not true multi-agent.

**Fix**: Invoke `/brainstorm` directly, not via delegation.

### "Takes longer than 60 seconds"

- Check network connection
- Try a simpler topic
- Verify parallel execution is working

## v2.0 Changes

This version corrects a critical architecture issue:
- **v1.0**: Assumed subagents could spawn more subagents (they can't)
- **v2.0**: Main session directly orchestrates parallel Task agents

See DESIGN.md for the full story.

## License

MIT - Use, modify, share freely.

## Credits

Synthesized by three agents (Pragmatist, Completionist, User Advocate) via the pattern this skill implements.
