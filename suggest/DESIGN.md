# Design Rationale: The Suggest Skill

## Synthesis Overview

This skill emerged from a three-agent brainstorm that balanced competing values:

| Agent | Focus | Core Value |
|-------|-------|------------|
| **Pragmatist** | Quick wins, actionable output | Simplicity, immediate utility |
| **Perfectionist** | Comprehensive analysis, structured format | Thoroughness, quality |
| **User Advocate** | Before/after diffs, apply mode | Clarity, actionability |

## Why `/suggest` as Companion to `/explain`

The brainstorm identified that `/explain` answers "what does this code do?" but leaves a gap: "how could it be better?"

**The synthesis insight**: Understanding without action is incomplete. A natural workflow is:
1. Explain → Understand the code
2. Suggest → See improvements
3. Apply → Implement changes

This creates a complete code review loop within Claude Code.

## How Each Perspective Shaped the Design

### From the Pragmatist

**Contributions**:
- Default limit of 5 suggestions (not 50)
- Quick depth option for fast scans
- High-impact, low-effort prioritization
- No ceremony - show the code, explain why, done

**Visible in**:
- `--limit 5` default (curated over comprehensive)
- `--depth quick` for lint-style checks
- Severity ranking: HIGH > MED > LOW
- One-sentence "Why" explanations

**Key argument**: "Users don't want a dissertation. They want to know what to fix and how."

### From the Perfectionist

**Contributions**:
- Structured severity levels
- Category filtering system
- Thorough depth option for comprehensive analysis
- Consistent output format

**Visible in**:
- `[HIGH]`, `[MED]`, `[LOW]` severity tags
- `--category bugs|security|performance|readability`
- `--depth thorough` for architectural suggestions
- Seven-part suggestion format (title, category, why, before, after)

**Key argument**: "Without structure, suggestions become noise. Severity and categories make output scannable and actionable."

### From the User Advocate

**Contributions**:
- Before/after diffs for every suggestion
- Interactive apply mode
- Clear one-sentence explanations
- Chaining with `/explain`

**Visible in**:
- Mandatory before/after code blocks
- `--apply` flag with step-by-step confirmation
- "Why" field in plain language
- Automatic context reuse from prior `/explain`

**Key argument**: "Show, don't tell. If you can't show the fix, don't suggest it."

## Key Design Decisions

### 1. Before/After Diffs Are Mandatory

**Problem**: Abstract suggestions ("improve error handling") aren't actionable.

**Solution**: Every suggestion must include concrete before/after code.

**Why not optional?**
- Vague suggestions waste time
- Diffs prove the suggestion is implementable
- Users can copy-paste directly
- `--apply` mode requires diffs to function

**Trade-off accepted**: Longer output, but dramatically more useful.

### 2. Limited Output (5 Suggestions Default)

**Pragmatist concern**: "50 suggestions = analysis paralysis"
**Perfectionist concern**: "What if there are 10 critical issues?"

**Solution**: Default to 5, allow `--limit N` up to 10.

**Rationale**:
- 5 suggestions is ~2 minutes of reading
- More suggestions = less likely any get implemented
- `--depth thorough` users can increase limit
- Critical issues surface first via severity ranking

**Evidence**: Code review studies show reviewers effectively process 5-7 items max.

### 3. Severity-First Ranking

**Algorithm**:
1. Score by impact (HIGH = critical, MED = notable, LOW = nice-to-have)
2. Score by effort (how hard to fix)
3. Rank: HIGH impact + LOW effort first
4. Apply filters (category, limit)

**Why not alphabetical or by line number?**
- Users want the most important issues first
- "Fix the crash before the typo"
- Enables confident early exit ("I'll just do the HIGH ones")

### 4. Category Filtering

**Four categories**:
- `bugs` - Runtime errors, edge cases, null checks
- `security` - Injection, auth, secrets, OWASP top 10
- `performance` - N+1 queries, blocking calls, missing memoization
- `readability` - Long functions, duplication, naming

**Why these four?**
- Cover 90% of code quality concerns
- Align with how developers think about issues
- Enable focused reviews ("just security today")
- Not too many (cognitive overload) or too few (useless filter)

**Rejected alternatives**:
- `style` - Too subjective, covered by linters
- `architecture` - Too broad, covered in `--depth thorough`
- `testing` - Separate concern, not code improvement

### 5. Interactive Apply Mode

**Problem**: Suggestions are useless if not implemented.

**Solution**: `--apply` flag steps through each suggestion with confirmation.

**Protocol**:
1. Show before/after diff
2. Ask: "Apply this change? (y/n/skip all)"
3. If yes, use Edit tool
4. Continue or stop based on response
5. Summarize at end

**Why not auto-apply?**
- Too risky - suggestions might be wrong
- User maintains control
- Educational - user sees each change
- Builds trust in the tool

**Why not batch apply?**
- Suggestions may conflict
- User might want some but not others
- Incremental is safer

### 6. Chaining with /explain

**Problem**: Running `/explain` then `/suggest` on same file is redundant.

**Solution**: If no target specified after `/explain`, reuse context.

**Example**:
```
/explain src/auth.ts --deep
/suggest                    # Automatically uses src/auth.ts + prior analysis
```

**Benefits**:
- Faster (no re-reading files)
- Smarter (knows what was explained)
- Natural workflow (understand → improve)

### 7. Depth Levels

**Three tiers**:
- `quick` - Surface-level, lint-style (~10 seconds)
- `standard` - Common issues + logic (~30 seconds, default)
- `thorough` - Architectural, cross-file (~60 seconds)

**Why three levels?**
- Quick for "did I forget anything obvious"
- Standard for regular reviews
- Thorough for major refactoring or audits

**Why not more granular?**
- Three is easy to remember
- Maps to user intent: fast/normal/deep
- More levels = analysis paralysis

## What We Didn't Build

### Rejected: Auto-fix Without Confirmation
**Idea**: Apply all suggestions automatically

**Why not**:
- Too risky - suggestions aren't perfect
- Removes user learning opportunity
- Could introduce bugs
- Trust must be earned incrementally

### Rejected: Severity Thresholds
**Idea**: Only show HIGH severity by default

**Why not**:
- MED issues are often valuable
- Users can filter with `--limit`
- Better to show 5 ranked than hide useful suggestions

### Rejected: File-Level Scoring
**Idea**: "This file scores 7/10 on code quality"

**Why not**:
- Arbitrary and demotivating
- Doesn't help fix anything
- Focus on actionable suggestions, not grades

### Rejected: Learning Mode
**Idea**: Explain each pattern in depth

**Why not**:
- That's what `/explain` is for
- Suggestions should be concise
- One-sentence "Why" is sufficient
- Chain with `/explain` for details

## Success Criteria

This skill succeeds if:

1. **Suggestions are applied** (not just read)
2. **Before/after diffs are clear** (copy-paste ready)
3. **Priority ranking is correct** (HIGHs are actually high)
4. **Apply mode is used** (workflow is trusted)
5. **Chaining with /explain feels natural**

## Lessons for Future Skills

### 1. Show, don't tell
Concrete examples (before/after) beat abstract advice every time.

### 2. Limit output by default
Curated beats comprehensive. Users can always ask for more.

### 3. Enable action
A suggestion without an apply mechanism is just noise.

### 4. Support workflows
Skills should chain together naturally (`explain → suggest → apply`).

### 5. Rank by impact
Users care about what matters most, not completeness.

## Open Questions

### Should suggestions reference line numbers?
**Current**: No line numbers in output
**Alternative**: Include line numbers for easier navigation
**Decision**: Deferred - before/after code is sufficient for now

### Should we support custom categories?
**Current**: Four fixed categories
**Alternative**: User-defined categories
**Decision**: Not in v1 - fixed categories cover 90% of needs

### Should apply mode support undo?
**Current**: No undo (use git)
**Alternative**: Built-in undo for recent applies
**Decision**: Not in v1 - git is the undo mechanism

## Conclusion

The `/suggest` skill synthesizes three perspectives:

- **Curated output** (Pragmatist) - 5 suggestions, ranked by impact
- **Structured format** (Perfectionist) - Severity, categories, consistent layout
- **Actionable results** (User Advocate) - Before/after diffs, apply mode

It's designed as the natural companion to `/explain`:
- Explain tells you what code does
- Suggest tells you what it could become
- Together they form a complete code understanding and improvement workflow

**The measure of success isn't how many issues are found—it's how many get fixed.**
