# Deliverables Summary

Complete production-ready `/verify` skill for Claude Code, enabling epistemic calibration for knowledge verification.

---

## Core Concept

**Discovery**: Not all knowledge ages equally.

This means:
- ✅ Stable knowledge (math, history, core CS) can be trusted from training
- ✅ Volatile knowledge (APIs, regulations, news) requires verification
- ✅ Claude shows reasoning about what was verified vs. trusted
- ✅ High-stakes queries get authoritative source pointers

---

## Files Delivered

### Core Skill Files

#### 1. **SKILL.md** (~2,000 words)
**Production skill specification for epistemic calibration**

- Complete frontmatter with metadata
- Knowledge Volatility Spectrum framework
- Decision Framework (when to verify vs. trust)
- Domain-specific guidelines (5 domains)
- Tool integration strategy (Context7, WebSearch, WebFetch)
- Example queries showing each pattern
- High-stakes query handling with disclaimers

**Status**: Production-ready

---

#### 2. **DESIGN.md** (~800 words)
**Design rationale documenting key decisions**

- Three-agent synthesis overview
- Key design decisions with trade-offs
- What was rejected and why
- Contributions from each agent
- Open questions for future development

---

#### 3. **README.md** (~600 words)
**User-facing documentation**

- What the skill does
- Quick start guide
- When to use vs. when not needed
- Feature overview
- Example invocations

---

#### 4. **EXAMPLES.md** (~1,200 words)
**Real-world usage scenarios**

Five comprehensive scenarios:
1. Technology query (Next.js setup)
2. Regulation query (GDPR requirements)
3. Mixed query (neural networks + latest architectures)
4. Stable query (quicksort algorithm)
5. High-stakes query (medical interactions)

Each shows knowledge assessment and output pattern.

---

#### 5. **INSTALL.md** (~500 words)
**Installation and troubleshooting guide**

- Installation options
- Verification steps
- Troubleshooting section
- Tool requirements (Context7, WebSearch, WebFetch)

---

#### 6. **LICENSE**
**MIT License**

Standard open source license for free use, modification, and distribution.

---

## Technical Specifications

### Skill Metadata

```yaml
name: verify
description: "Epistemic calibration — knowing when to verify vs. trust training knowledge"
version: 1.1.0
author: PSthelyBlog
tags: [knowledge, verification, accuracy, current-information, metacognition]
model: sonnet
```

### Knowledge Volatility Spectrum

```
STABLE ◄─────────────────────────────────────────────► VOLATILE

Math        Core CS       Established      Evolving        Current
Logic       Algorithms    Standards        Best Practices  Events
History     Language      Classic          Active          News
Philosophy  Fundamentals  Literature       Research        Releases
```

### Tool Integration

| Tool | Use Case |
|------|----------|
| Context7 | Technology documentation (preferred) |
| WebSearch | General current information |
| WebFetch | Specific authoritative sources |

---

## Output Format

Every verify response includes:

```markdown
## Knowledge Assessment

**Verified (volatile/time-sensitive):**
- [What was looked up and why]

**Trusted training (stable knowledge):**
- [What was used from training and why]

**Confidence notes:**
- [Any remaining uncertainties]
```

---

## File Sizes

| File | Words | Purpose |
|------|-------|---------|
| SKILL.md | ~2,000 | Core skill specification |
| DESIGN.md | ~800 | Design rationale |
| EXAMPLES.md | ~1,200 | Real-world scenarios |
| README.md | ~600 | User documentation |
| INSTALL.md | ~500 | Installation guide |
| DELIVERABLES.md | ~400 | This file |
| **Total** | **~5,500** | Complete skill package |

---

## Quality Assurance

### Production-Ready Checklist

- ✅ SKILL.md with valid frontmatter
- ✅ Knowledge Volatility Spectrum defined
- ✅ Clear verify vs. trust decision framework
- ✅ Domain-specific guidelines (5 domains)
- ✅ Tool integration documented
- ✅ High-stakes query handling with disclaimers
- ✅ Real-world examples
- ✅ Installation guide
- ✅ No placeholders or TODOs
- ✅ MIT License

### Testing Recommendations

Before deployment, verify:

1. **Technology query**: `/verify set up Next.js with auth` (should verify)
2. **Stable query**: `/verify explain quicksort` (should trust training)
3. **Mixed query**: `/verify neural networks + latest architectures` (should do both)
4. **High-stakes query**: `/verify drug interactions` (should verify + disclaim)

---

## Success Metrics

A verify response:
- ✅ Correctly identifies what might be stale
- ✅ Verifies volatile information from current sources
- ✅ Trusts training for genuinely stable knowledge
- ✅ Shows reasoning about epistemic confidence
- ✅ Points to authoritative sources for high-stakes topics
- ✅ Acknowledges remaining uncertainties

---

## Final Status

**PRODUCTION-READY**

The skill provides epistemic calibration—teaching Claude when to verify vs. trust training knowledge. The Knowledge Volatility Spectrum and domain-specific guidelines ensure accurate, efficient responses.

Copy `verify/` directory to Claude Code skills folder and invoke `/verify [query]`.

---

**Total Deliverables**: 6 files, ~5,500 words, production-ready skill for epistemic calibration
