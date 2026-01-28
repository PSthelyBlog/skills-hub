# Skills Hub for Claude Code

**One command to access all skills — intelligent routing, seamless chaining.**

## What It Does

The `/skills-hub` command is the unified entry point for all Claude Code skills. Instead of remembering which skill does what and how to invoke each one, just describe what you want — the hub routes your request to the right skill(s) automatically.

## Quick Start

```bash
# Explicit skill invocation
/skills-hub explain src/api/auth.ts
/skills-hub suggest --category security src/

# Natural language routing
/skills-hub what does the authentication middleware do
/skills-hub how can I improve the database layer

# Chain skills together
/skills-hub explain src/api.ts then suggest improvements

# Run independent skills in parallel
/skills-hub explain auth.ts and explain users.ts
```

## Installation

1. Copy the `skills-hub` directory to your Claude Code skills folder
2. Ensure companion skills are also installed (explain, suggest, etc.)
3. Restart Claude Code or reload skills
4. Type `/skills-hub` to start using it

## Available Skills

| Skill | Purpose | When to Use |
|-------|---------|-------------|
| **explain** | Understand code — quick answers to deep dives | "What does this do?", "How does X work?" |
| **suggest** | Actionable code improvements with diffs | "How can this be better?", "Review this code" |
| **brainstorm** | Multi-perspective debates (3 AI agents) | Design decisions, architecture, trade-offs |
| **update** | Modernize code with fresh documentation | Outdated deps, deprecated APIs, migrations |
| **grounded** | Verify volatile knowledge, trust stable | Current events, new APIs, time-sensitive info |
| **write-for-claude** | Generate Claude-optimized instructions | CLAUDE.md, SKILL.md, AGENTS.md, prompts |

## Core Features

### Intelligent Routing

The hub understands intent, not just syntax:

```bash
# These all route to explain
/skills-hub what does getUserById do
/skills-hub how does authentication work
/skills-hub why is the cache invalidating

# These all route to suggest
/skills-hub how can I improve src/api.ts
/skills-hub review the database layer
/skills-hub find bugs in the payment code
```

### Skill Chaining

Connect skills with "then" for sequential workflows:

```bash
# Understand, then improve
/skills-hub explain src/api/users.ts then suggest improvements

# Decide, then implement
/skills-hub brainstorm caching strategy then suggest implementation

# Verify, then modernize
/skills-hub grounded React Query v5 patterns then update src/hooks/
```

### Parallel Execution

Run independent skills concurrently:

```bash
# Both explanations run in parallel
/skills-hub explain auth.ts and explain payments.ts

# Independent suggestions run concurrently
/skills-hub suggest src/api/ and suggest src/utils/
```

### Flag Passthrough

All flags pass directly to the target skill:

```bash
/skills-hub explain auth flow --deep
/skills-hub suggest --category security --depth thorough
/skills-hub update --dry-run --max-risk low src/
```

## Example Session

```
You: /skills-hub what does the payment processor do

Hub routes to: explain

Claude:
## processPayment

**What it does**: Handles credit card charges via Stripe API

**Key points**:
- Validates amount and currency before charging
- Returns transaction ID on success, null on failure
- Automatically retries once on network errors

**Why it matters**: Safe to call without try-catch, but check for null returns.

---

You: /skills-hub how can I improve it

Hub routes to: suggest

Claude:
## Suggestions for processPayment

### 1. [HIGH] Add rate limiting
**Category:** security
...

---

You: /skills-hub brainstorm error handling strategy then write-for-claude document it

Hub chains: brainstorm → write-for-claude

Claude:
## Workflow: brainstorm → write-for-claude

### brainstorm: Error Handling Strategy
[Multi-perspective debate output]

### write-for-claude: Documentation
[Generated CLAUDE.md section]
```

## Design Philosophy

### Invisible Infrastructure

The hub succeeds when you forget it exists. You should feel like you're working with one intelligent system that happens to have multiple capabilities.

### Intent Over Syntax

Think in terms of goals ("make this better"), not tools ("invoke suggest"). The hub translates your intent to the right skill.

### Context Preservation

Skills execute as Task subagents, keeping your main context clean. Only results flow back, not execution traces.

### Graceful Chaining

Context flows between chained skills automatically. The output of one skill informs the next.

## Tips

### When to Use Explicit Names

- You know exactly which skill you need
- Intent parsing might be ambiguous
- You want predictable routing

### When to Use Natural Language

- You're not sure which skill applies
- Your request naturally describes the goal
- You want the hub to decide

### Effective Chaining

**Good chains:**
- explain → suggest (understand before improving)
- brainstorm → suggest (decide then implement)
- grounded → update (verify then modernize)
- brainstorm → write-for-claude (decide then document)

**Avoid:**
- Long chains (>3 skills) — break into separate requests
- Circular chains (suggest → explain → suggest)
- Incompatible outputs (some skills don't chain well)

### Special Note: brainstorm

The brainstorm skill spawns its own Task agents (3 personas debating). It cannot be delegated to a subagent. The hub handles this automatically, executing brainstorm directly while delegating other skills.

## Performance

- **Single skill**: Subagent spin-up + skill execution
- **Chained skills**: Sequential execution with context passing
- **Parallel skills**: Concurrent execution for independent targets

Using the hub adds minimal overhead — the context isolation benefits outweigh the cost.

## Contributing

Improvements welcome:
- Better intent patterns
- New skill integrations
- Enhanced chaining logic
- Performance optimizations

See `DESIGN.md` for architecture details.

## License

MIT License — use, modify, share freely.

## Credits

Part of the Claude Code Skills Hub collection. Built with multi-agent synthesis patterns and Task-based orchestration.
