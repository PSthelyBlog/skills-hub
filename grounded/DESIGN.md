# Design Rationale: The Grounded Skill

## Synthesis Overview

This skill emerged from a three-agent brainstorm that debated how to design `/grounded`:

| Agent | Position | Core Value |
|-------|----------|------------|
| **Pragmatist** | Simple volatility check, minimal verification | Speed, "don't slow me down" |
| **Perfectionist** | Comprehensive verification, multiple sources | Accuracy, authoritative sources |
| **User Advocate** | Transparent reasoning, clear confidence levels | Trust, understanding why |

## Key Design Decisions

### 1. Volatility-Based Decision Framework

**Problem**: When should Claude verify vs. trust training knowledge?

**Pragmatist wanted**: "Just check anything with dates or versions"
**Perfectionist wanted**: "Verify everything, better safe than sorry"

**Solution**: Knowledge Volatility Spectrum with clear categories

**Why**:
- Not all knowledge ages equally
- Checking everything is slow and unnecessary
- Blind trust in training leads to stale information
- Clear categories reduce decision overhead

**Trade-off accepted**: Some edge cases require judgment calls.

### 2. Show Your Reasoning (Knowledge Assessment)

**Problem**: How transparent should the verification process be?

**Pragmatist wanted**: "Just give the answer, hide the plumbing"
**User Advocate wanted**: "Show exactly what was verified and why"

**Solution**: Mandatory Knowledge Assessment section showing what was verified vs. trusted

**Why**:
- Users can calibrate their trust
- Educational—users learn what's volatile
- Debuggable—clear why something might be wrong
- Builds confidence in the process

### 3. Domain-Specific Guidelines

**Problem**: How detailed should verification guidance be?

**Pragmatist wanted**: "Generic rules apply everywhere"
**Perfectionist wanted**: "Custom rules for every domain"

**Solution**: Five key domains with specific verify/trust guidance:
- Technology & Software
- Regulations & Compliance
- Science & Medicine
- News & Current Events
- Products & Services

**Why**:
- Covers 90% of real queries
- Specific enough to be actionable
- General enough to not explode in complexity
- Each domain has genuinely different volatility patterns

### 4. Tool Integration Strategy

**Problem**: Which tools should be used for verification?

**Solution**: Tiered approach based on query type:
- **Context7**: Preferred for technology/frameworks (most accurate for code)
- **WebSearch**: General current information, news, regulations
- **WebFetch**: Specific authoritative sources when URL known

**Why**:
- Context7 is purpose-built for documentation accuracy
- WebSearch covers breadth
- WebFetch for depth on known sources
- Multiple tools increase reliability

### 5. High-Stakes Query Handling

**Problem**: How to handle queries where being wrong matters?

**Perfectionist wanted**: "Refuse to answer medical/legal questions"
**User Advocate wanted**: "Help but with strong disclaimers"

**Solution**: Answer with verification + mandatory disclaimers + authoritative source pointers

**Why**:
- Users need help even with sensitive topics
- Disclaimers set appropriate expectations
- Pointing to authoritative sources is genuinely helpful
- Refusing entirely is user-hostile

---

## What We Rejected

### Rejected: Always Verify Everything

**Why not**: Slow, expensive, unnecessary for stable knowledge. Checking "how does quicksort work" is wasteful.

### Rejected: Never Verify Anything

**Why not**: Training has a cutoff. Framework APIs change. Regulations update. Users deserve current information.

### Rejected: Binary Verify/Trust Categories

**Why not**: Reality is a spectrum. "Best practices" are mostly stable but evolve at the edges.

### Rejected: Hide Verification Process

**Why not**: Users can't calibrate trust if they don't see what was checked. Transparency builds confidence.

---

## Contribution From Each Agent

### Pragmatist Contributions
- Volatility spectrum (not everything needs checking)
- Stable knowledge categories (math, history, core CS)
- "Five Questions" quick checklist
- Anti-patterns section (don't over-verify)

### Perfectionist Contributions
- Domain-specific guidelines
- Multiple tool integration
- High-stakes query handling
- Authoritative source requirements

### User Advocate Contributions
- Knowledge Assessment section (show reasoning)
- Confidence notes in output
- Clear volatility signals list
- Educational examples for each pattern

---

## The Meta-Principle

**Calibrated confidence beats false certainty.**

The skill teaches Claude to:
1. Recognize when knowledge might be stale
2. Verify volatile information efficiently
3. Trust stable knowledge confidently
4. Show reasoning transparently
5. Point to authoritative sources

This produces responses that are:
- Accurate for current information
- Efficient for stable knowledge
- Transparent about confidence
- Helpful even for sensitive topics

---

## Open Questions

### Should there be a "force verify" flag?

Users might want to force verification even for stable knowledge ("I know this is basic, but double-check anyway").

**Current status**: Not implemented. Users can phrase query to trigger verification.

### Should verification results be cached?

Multiple queries about the same topic could reuse verification results.

**Current status**: Not implemented. Each query verifies fresh.

### Could we auto-detect more volatility signals?

ML-based detection of query volatility rather than keyword matching.

**Current status**: Keyword/pattern based. Works well enough.

---

## Conclusion

The grounded skill addresses a fundamental challenge: Claude's training has a cutoff, but not all knowledge ages. The skill provides:

- **Clear framework** for when to verify vs. trust
- **Domain-specific** guidance for common query types
- **Transparent reasoning** so users understand confidence
- **Tool integration** for accurate verification
- **High-stakes handling** with appropriate disclaimers

The result is responses that are current when currency matters, efficient when it doesn't, and transparent about the difference.
