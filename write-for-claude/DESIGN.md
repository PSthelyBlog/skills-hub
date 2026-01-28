# Design Rationale: The Write-For-Claude Skill

## Synthesis Overview

This skill emerged from a three-agent brainstorm that debated how to design `/write-for-claude`:

| Agent | Position | Core Value |
|-------|----------|------------|
| **Pragmatist** | Simple templates, quick transforms | Speed, "just fix my text" |
| **Perfectionist** | Comprehensive rules, validation checklist | Precision, zero ambiguity |
| **User Advocate** | Show transformations, teach principles | Learning, transparency |

## Key Design Decisions

### 1. MUST/SHOULD/MAY Directive System

**Problem**: How should instructions be prioritized?

**Pragmatist wanted**: "Just use 'do this' and 'don't do that'"
**Perfectionist wanted**: "RFC 2119 full compliance with SHALL, REQUIRED, etc."

**Solution**: Three-level system: MUST, SHOULD, MAY (plus MUST NOT)

**Why**:
- Three levels cover all real needs
- More levels create confusion
- RFC 2119 is familiar to developers
- Clear hierarchy: non-negotiable → preference → optional

**Trade-off accepted**: Less nuance than full RFC 2119, but more usable.

### 2. Explicit Trigger Conditions

**Problem**: When do directives apply?

**Pragmatist wanted**: "Assume directives always apply"
**Perfectionist wanted**: "Every directive needs explicit scope"

**Solution**: Require WHEN/IF/FOR prefix OR universal scope declaration

**Why**:
- "MUST: Use TypeScript" is ambiguous (all files? new files only?)
- "FOR all files in src/: MUST use TypeScript" is clear
- Prevents Claude from guessing at scope
- Makes edge cases explicit

### 3. Decision Trees Over Judgment

**Problem**: How to handle conditional logic?

**User Advocate wanted**: "Let Claude use good judgment"
**Perfectionist wanted**: "Enumerate every case"

**Solution**: Convert "use judgment" to explicit decision trees with ELSE clauses

**Why**:
- "Good judgment" means different things to different Claude sessions
- Decision trees produce consistent behavior
- Edge cases surface during tree construction
- ELSE clause prevents undefined behavior

### 4. Anti-Pattern Examples

**Problem**: How to show what NOT to do?

**Pragmatist wanted**: "Just show the right way"
**User Advocate wanted**: "Show wrong AND right for contrast"

**Solution**: Side-by-side WRONG/RIGHT examples in Anti-Patterns section

**Why**:
- Users often have the "wrong" pattern in their existing docs
- Seeing the contrast teaches the principle
- Makes it easy to find and fix issues
- Demonstrates the transformation

### 5. Optimization Report

**Problem**: Should output explain what was changed?

**Pragmatist wanted**: "Just give the final output"
**User Advocate wanted**: "Show a detailed transformation log"

**Solution**: Brief "Optimization Applied" section at end showing key transformations

**Why**:
- Users learn what was improved
- Transparent about changes made
- Helps users write better content next time
- Not so detailed it overwhelms

---

## What We Rejected

### Rejected: Free-Form Transformation

**Why not**: Without structure, output varies between sessions. The whole point is consistency.

### Rejected: Full RFC 2119 Compliance

**Why not**: REQUIRED vs MUST vs SHALL creates confusion. Three levels suffice.

### Rejected: Automated Validation Only

**Why not**: Users need to understand WHY changes were made, not just see the result.

### Rejected: Domain-Specific Templates

**Why not**: Four output types (CLAUDE.md, SKILL.md, AGENTS.md, Prompt) cover all needs. More templates = more complexity.

---

## Contribution From Each Agent

### Pragmatist Contributions
- Four output types (not dozens of templates)
- Simple substitution tables
- Quick transformation rules
- Single validation checklist

### Perfectionist Contributions
- Hedge word elimination list
- Decision tree requirements (ELSE clauses)
- Definitions section requirement
- Comprehensive anti-pattern examples

### User Advocate Contributions
- "Optimization Applied" transparency section
- WRONG/RIGHT comparison format
- Validation test at end
- Placeholder markers for missing info

---

## The Core Principle

**Unambiguous instructions produce consistent behavior.**

Two different Claude sessions, reading the same CLAUDE.md or SKILL.md, should behave identically. This requires:

1. **Explicit directives** (MUST/SHOULD/MAY)
2. **Clear triggers** (WHEN/IF/FOR)
3. **Defined terms** (no ambiguous vocabulary)
4. **Complete decision trees** (ELSE clauses)
5. **Enumerated edge cases** (DEFAULT handling)

The skill transforms vague prose into structured instructions that pass the "identical behavior" test.

---

## Meta-Compliance

This skill specification follows its own rules:
- Uses MUST/SHOULD/MAY throughout
- Every directive has trigger or universal scope
- Decision trees have ELSE/DEFAULT
- Terms defined in context
- Edge cases enumerated
- No hedge words
- Examples show correct vs incorrect

---

## Open Questions

### Should there be a "lint" mode?

Check existing documents for violations without transforming them.

**Current status**: Validation checklist serves this purpose. Full lint mode not implemented.

### Could we support custom directive keywords?

Some teams use REQUIRED, SHALL, etc.

**Current status**: Standardized on MUST/SHOULD/MAY. Conversion from other keywords could be added.

### Should output include confidence scores?

Rate how unambiguous the generated content is.

**Current status**: Validation checklist is binary (pass/fail). Scores not implemented.

---

## Conclusion

The write-for-claude skill solves a fundamental problem: Claude interprets vague instructions inconsistently. By transforming prose into structured directives with explicit triggers, defined terms, and complete decision trees, the skill ensures that any Claude session reading the output will behave identically.

The result is documentation that is:
- **Unambiguous**: One interpretation, not many
- **Complete**: Edge cases and defaults handled
- **Consistent**: Same behavior across sessions
- **Transparent**: Users see what was optimized
