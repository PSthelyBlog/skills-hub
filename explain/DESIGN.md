# Design Rationale: The Explain Skill

## Synthesis Overview

This skill emerged from a three-agent brainstorm that balanced competing values:

| Agent | Proposed Skill | Core Value |
|-------|---------------|------------|
| **Pragmatist** | `/cleanup` | Simplicity, immediate utility, low friction |
| **Completionist** | `/analyze-performance` | Comprehensive analysis, depth, thoroughness |
| **User Advocate** | `/explain` | Learning, understanding, progressive teaching |

## Why `/explain` Won

In Round 2, all three agents converged on `/explain` as the most universally valuable concept:

- **Pragmatist**: "Teaching compounds; execution is one-time"
- **Completionist**: "Every skill should teach, not just execute"
- **User Advocate**: "This is THE skill Claude Code needs most"

The consensus wasn't just about which skill to build, but about a **design philosophy**: skills should build user capability, not create dependency.

## How Each Perspective Shaped the Design

### From the Pragmatist

**Contributions**:
- Default to fast, concise answers (Haiku model)
- No ceremony - answer the question directly
- Offer depth as opt-in, not default
- Keep response time under 30 seconds for common queries

**Visible in**:
- Quick Explain format (3 bullet points max)
- Haiku-first model selection
- "Need more? Add `--deep`" footer
- Minimal context gathering by default

### From the Completionist

**Contributions**:
- Deep mode for comprehensive analysis
- Structured investigation protocol
- Coverage of dependencies, trade-offs, gotchas
- Multi-dimensional understanding (what, how, why, when)

**Visible in**:
- `--deep` flag and comprehensive format
- Seven-section deep explanation structure
- Progressive context gathering
- Explicit design choices and trade-offs section

### From the User Advocate

**Contributions**:
- Teaching-first orientation
- Progressive disclosure (layered depth)
- Plain language before technical terms
- Context awareness and calibration

**Visible in**:
- "Teaching Principles" section
- Examples from user's actual codebase
- Natural next-step suggestions
- Error handling with helpful redirects

## Key Design Decisions

### 1. The `--deep` Flag

**Problem**: Users have different needs at different times. Sometimes you want "what does this do?" in 10 seconds. Sometimes you need to understand architecture before making changes.

**Solution**: Two-tier system with progressive disclosure.

**Why not three tiers?** (quick/medium/deep)
- Adds cognitive load (which do I choose?)
- Medium tier rarely used (people want fast OR comprehensive)
- Two-tier maps to mental model: "quick answer" vs "I need to understand this"

**Implementation**:
- Default = Haiku, <30 second read
- `--deep` = Sonnet if needed, comprehensive analysis
- Automatic escalation if topic proves complex

### 2. Model Selection Strategy

**Pragmatist concern**: "Don't waste time/money on Sonnet for simple queries"
**Completionist concern**: "Don't compromise quality on complex analysis"

**Solution**: Smart defaults with automatic escalation.

```
Start with Haiku for:
- Single function lookups
- Simple "what does X do"
- Error message explanations

Escalate to Sonnet for:
- --deep flag present
- Architecture questions
- Cross-cutting analysis
- Mid-explanation complexity discovery
```

**Cost impact**:
- 80% of queries stay in Haiku (10x cheaper, 5x faster)
- 20% get Sonnet depth when it matters
- User can force depth with `--deep`

### 3. Response Structure

**Two formats, not a spectrum**:

**Quick Format** (default):
```
What it does: [one line]
Key points: [3 bullets]
Why it matters: [practical context]
---
Need more? Add --deep
```

**Deep Format** (--deep):
```
Overview: [comprehensive summary]
How it works: [detailed breakdown]
Dependencies: [what relies on what]
Design choices: [why this way]
Common patterns: [broader context]
Gotchas: [edge cases]
Related: [exploration paths]
```

**Why not customize per query?**
- Consistent structure builds user mental models
- Clear expectations (quick vs deep)
- Easy to skim/scan for specific info

### 4. Teaching Principles Integration

All three agents agreed: **teach, don't just execute**.

**Implemented via**:
1. **Plain language first**: "Fetches a user" not "Executes SELECT query with JOIN"
2. **Concrete examples**: From THEIR code, not generic samples
3. **Progressive terms**: Introduce technical concepts after plain explanation
4. **Practical implications**: "Why it matters" in every quick explain

**Example**:
```
❌ "Uses bcrypt hashing algorithm with default cost factor"
✅ "Passwords are encrypted - safe to store, can't be reversed"
   (with --deep: "Uses bcrypt with cost factor 10...")
```

### 5. Context Awareness

**User Advocate insight**: "Meet users where they are"

**Implementation**:
- Check recent file reads (are they already exploring this?)
- Note current directory (tests? src? docs?)
- Adapt language to apparent expertise level
- Offer calibrated next steps

**Example**:
```
User in /tests/auth/ asks: "/explain login flow"
→ Emphasize test coverage, mock patterns, edge cases

User in /src/controllers/ asks same question
→ Emphasize implementation, dependencies, security
```

## What We Didn't Build

### Rejected: Multi-skill suite
**Completionist wanted**: `/explain` + `/cleanup` + `/analyze-performance` as complementary suite

**Why not**:
- One polished skill > three rough ones
- `/explain` can guide cleanup and performance (teaching > doing)
- Scope creep vs. production-ready trade-off

**Future**: Other skills can adopt the same teaching philosophy

### Rejected: Automatic depth detection
**Idea**: Claude decides when to go deep without user flag

**Why not**:
- Unpredictable response time (bad UX)
- User loses control
- Hard to calibrate "is this complex enough?"
- Better to give user the choice

### Rejected: Interactive follow-up
**Idea**: "Want me to explain X now?" conversational prompts

**Why not**:
- Adds friction for users who know what they want
- Makes simple queries chatty
- Better to offer next steps, let user drive

## Success Criteria

This skill succeeds if:

1. **80%+ of queries use default mode** (Pragmatist: simplicity wins)
2. **Deep mode feels comprehensive** (Completionist: depth when needed)
3. **Users learn, don't just get answers** (User Advocate: teaching works)
4. **Response time <30sec for quick, <2min for deep** (Pragmatist: respect time)
5. **Users invoke it before modifying unfamiliar code** (All: it's becoming a habit)

## Lessons for Future Skills

### 1. Progressive disclosure is powerful
Don't force users to choose between "too simple" and "overwhelming". Give them layers.

### 2. Model selection matters
Haiku for speed, Sonnet for depth. Let the task drive the model, not vice versa.

### 3. Teaching > Executing
A skill that builds understanding has compounding value. One that just executes doesn't.

### 4. Structure aids scanning
Consistent formats let users find what they need quickly. Bespoke responses require full reading.

### 5. Default to less
You can always offer more. Starting heavy and scaling down is jarring.

## Open Questions

### Should --deep be --verbose or --full or --comprehensive?
- `--deep` feels developer-friendly
- `--full` might be clearer
- `--verbose` has CLI precedent

**Decision**: `--deep` - short, clear, implies depth vs breadth

### Should we support --format json or --format markdown?
**Deferred**: Start with human-readable defaults. Add machine-readable formats if users request.

### Should explanations cite line numbers?
**Yes for deep mode**, **no for quick mode**. Quick explanations should feel conversational, not like compiler output.

### Should we cache explanations?
**Not in v1**. Codebase changes, explanations go stale. Better to regenerate quickly (Haiku makes this viable).

## Conclusion

The `/explain` skill synthesizes three perspectives into a coherent whole:

- **Simple by default** (Pragmatist)
- **Deep when needed** (Completionist)
- **Teaching always** (User Advocate)

It's not a compromise - it's an integration. Each perspective made the skill stronger:

- Pragmatist kept it fast and practical
- Completionist ensured depth was available
- User Advocate made it genuinely helpful

The result: a skill that respects user time while building user capability.

**That's the synthesis in action.**
