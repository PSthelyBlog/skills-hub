# Skills Hub Design

## Problem Statement

Users have access to multiple specialized skills (explain, suggest, brainstorm, update, grounded, write-for-claude), but:

1. **Discovery overhead** - Users must know which skill exists for their need
2. **Syntax friction** - Each skill has different invocation patterns
3. **Context loss** - Switching between skills loses valuable context
4. **Manual orchestration** - Chaining skills requires explicit multi-step work
5. **Token waste** - Each skill interaction consumes main context

## Solution: The Skills Hub

A meta-skill that acts as an intelligent router and orchestrator.

```
┌─────────────────────────────────────────────────────────────┐
│                        User Request                          │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                        Skills Hub                            │
│  ┌─────────────────────────────────────────────────────┐    │
│  │  1. Parse intent                                     │    │
│  │  2. Route to skill(s)                               │    │
│  │  3. Detect chaining                                  │    │
│  │  4. Execute via subagents                           │    │
│  │  5. Synthesize results                              │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
           │              │              │              │
           ▼              ▼              ▼              ▼
     ┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐
     │ Task    │    │ Task    │    │ Task    │    │ Task    │
     │ explain │    │ suggest │    │ brainstorm│   │ update  │
     └─────────┘    └─────────┘    └─────────┘    └─────────┘
           │              │              │              │
           └──────────────┴──────────────┴──────────────┘
                              │
                              ▼
                    ┌─────────────────┐
                    │ Synthesized     │
                    │ Result          │
                    └─────────────────┘
```

## Key Design Decisions

### 1. Task-Based Delegation

**Decision:** Execute each skill via the Task tool with `subagent_type="general-purpose"`.

**Exception: brainstorm cannot be delegated.** The brainstorm skill spawns 3 parallel Task agents internally. Since subagents cannot invoke the Task tool (to prevent infinite recursion), brainstorm must be executed directly by the main session, not wrapped in a Task call.

**Rationale (for delegatable skills):**
- **Token efficiency** - Subagent execution doesn't consume main context
- **Clean isolation** - Each skill runs with focused context
- **Parallel execution** - Independent skills can run concurrently
- **Error containment** - Skill failures don't crash the main session

**Trade-off accepted:** Slightly higher latency for subagent spin-up, but worth it for context preservation.

### 2. Intent-Based Routing

**Decision:** Infer skill from request semantics, not just explicit naming.

**Rationale:**
- Users think in terms of goals ("make this better"), not tools ("invoke suggest")
- Reduces learning curve for skill system
- Enables natural language interaction

**Implementation:**
```
"what does this do" → explain
"how can this be improved" → suggest
"should we use X or Y" → brainstorm
```

### 3. Explicit Chaining Syntax

**Decision:** Support "then" keyword for skill chaining.

**Rationale:**
- Natural language pattern ("do X then Y")
- Unambiguous parsing
- Familiar workflow pattern

**Examples:**
```
explain → suggest  (understand then improve)
brainstorm → write-for-claude  (decide then document)
grounded → update  (verify then modernize)
```

### 4. Parallel Execution When Possible

**Decision:** Launch independent Tasks in parallel.

**Rationale:**
- Significant latency reduction (parallel vs sequential)
- Task tool supports multiple concurrent invocations
- No data dependency means safe parallelization

**Detection:** If chaining keyword absent AND multiple targets, execute parallel.

### 5. Minimal Result Wrapping

**Decision:** Return skill output with minimal hub-added structure.

**Rationale:**
- Users want skill results, not wrapper boilerplate
- Each skill already has defined output format
- Hub should be transparent infrastructure

## Architecture: Why Subagents?

### The Context Window Problem

Without subagents:
```
Main Context Window
├── System prompt
├── Conversation history
├── Skill instructions (loaded)
├── Skill execution context
├── Skill output
└── Next skill instructions (loaded)
    └── Previous skill output still here
        └── Context bloat accumulates
```

With subagents:
```
Main Context Window
├── System prompt
├── Conversation history
├── Hub routing logic
└── Skill results only (summarized)

Subagent 1 (isolated)        Subagent 2 (isolated)
├── Skill 1 instructions     ├── Skill 2 instructions
├── Skill 1 execution        ├── Context from skill 1
└── Output → main            └── Output → main
```

### Benefits

1. **Token efficiency** - Main context only holds results, not execution traces
2. **Skill isolation** - Each skill runs in clean environment
3. **Parallel scaling** - Multiple skills can execute simultaneously
4. **Error recovery** - Failed skill doesn't corrupt main context
5. **Composability** - Easy to add new skills without modifying hub

## Skill Compatibility Matrix

| First Skill | → explain | → suggest | → brainstorm | → update | → grounded | → write-for-claude |
|-------------|-----------|-----------|--------------|----------|------------|-------------------|
| explain     | -         | ✓ Common  | ✓ Good       | △ Rare   | △ Rare     | ✓ Good            |
| suggest     | △ Rare    | -         | △ Rare       | △ Rare   | △ Rare     | ✓ Good            |
| brainstorm  | △ Rare    | ✓ Good    | -            | △ Rare   | △ Rare     | ✓ Common          |
| update      | △ Rare    | ✓ Good    | △ Rare       | -        | △ Rare     | △ Rare            |
| grounded    | ✓ Good    | △ Rare    | ✓ Good       | ✓ Common | -          | ✓ Good            |
| write-for-claude | △ Rare | △ Rare   | △ Rare       | △ Rare   | △ Rare     | -                 |

Legend: ✓ Common/recommended, △ Possible but uncommon, - Same skill

**Note on brainstorm chains:** Since brainstorm cannot be delegated (it spawns its own Task agents), chains involving brainstorm require special execution:
- `X → brainstorm`: Delegate X via Task, then execute brainstorm directly
- `brainstorm → Y`: Execute brainstorm directly, then delegate Y via Task

## Error Handling Philosophy

### Graceful Degradation

```
IF skill fails:
  1. Inform user of specific failure
  2. Offer alternatives:
     - Retry with different parameters
     - Use different skill
     - Continue without failed skill (in chains)
  3. Never leave user stranded
```

### No Silent Failures

```
MUST NOT: Silently skip failed skills
MUST: Report what failed and why
MUST: Offer actionable next steps
```

## Future Extensibility

### Adding New Skills

1. Create `./skills-hub/[skill-name]/SKILL.md`
2. Add entry to hub's skill table
3. Add intent patterns for routing
4. Add to compatibility matrix

The hub doesn't need code changes — just documentation updates.

### Custom Skill Chains

Future enhancement: Named chain presets
```
/skills-hub workflow:code-review src/api.ts
→ Executes: explain → suggest → (optional) apply
```

### Skill Parameters Passthrough

All flags and parameters after skill name pass through unchanged:
```
/skills-hub suggest --category security --depth thorough src/
→ suggest receives: --category security --depth thorough src/
```

## Design Principles

1. **Invisible infrastructure** - Users shouldn't feel like they're managing tools
2. **Intent over syntax** - Understand what user wants, not just what they typed
3. **Parallel when possible** - Optimize for speed
4. **Isolated execution** - Protect main context
5. **Graceful chains** - Pass context cleanly between skills
6. **Fail informatively** - Never silent failures

---

**The hub succeeds when users forget it exists.** They should feel like they're working with one intelligent system that happens to have multiple capabilities.
