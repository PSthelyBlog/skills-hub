# Verify Skill

---
name: verify
description: "Verify volatile knowledge, trust stable knowledge — epistemic calibration for accurate responses"
version: 1.3.0
author: PSthelyBlog
tags: [knowledge, verification, accuracy, current-information, metacognition]
model: sonnet
examples:
  - "/verify create a Next.js app with authentication"
  - "/verify what are the current EU AI regulations"
  - "/verify explain the latest treatment guidelines for Type 2 diabetes"
  - "/verify how do I configure AWS Lambda with the newest runtime"
---

## Trigger

WHEN user invokes `/verify [query]`: Execute this protocol.

## Protocol

### Step 1: Analyze Query

Extract from user input:

| Component | How to identify |
|-----------|-----------------|
| Domain | Technology, regulation, science, news, products, or stable knowledge |
| Time sensitivity | Presence of volatility signals (see below) |
| Stakes | Medical/legal/financial = critical; general info = standard |

#### 1.1 Volatility Signals

IF query contains ANY of these terms: MUST verify

| Signal type | Examples |
|-------------|----------|
| Recency markers | "latest", "current", "now", "recent", "new", "updated" |
| Version references | "v15", "version 3", "newest", specific version numbers |
| Date references | Year numbers, "2024", "2025", date ranges |
| Change indicators | "changed", "deprecated", "replaced", "new API" |

### Step 2: Classify Knowledge Type

FOR each aspect of the query, classify as VERIFY or TRUST:

#### 2.1 VERIFY (Use WebSearch, Context7, WebFetch)

| Domain | Verify when |
|--------|-------------|
| Technology | Package versions, API syntax, cloud configurations, security practices |
| Regulations | Specific requirements, deadlines, tax rates, compliance standards |
| Science/Medicine | Treatment guidelines, drug interactions, recent research |
| News/Events | Post-cutoff developments, company status, political changes |
| Products/Services | Pricing, features, availability, API limits |
| Statistics | Numbers, data points, metrics, demographics |

#### 2.2 TRUST TRAINING (No verification needed)

| Domain | Trust when |
|--------|------------|
| Mathematics | Proofs, formulas, algorithms, complexity analysis |
| Core CS | Data structures, sorting algorithms, design patterns |
| History | Historical facts, established events, biographical data (pre-cutoff) |
| Philosophy | Ethical frameworks, logical reasoning, established theories |
| Language fundamentals | Core syntax, grammar, basic programming constructs |
| Physics/Chemistry | Laws, periodic table, mechanics, thermodynamics |

#### 2.3 Decision Tree

```
IF query contains volatility signal: VERIFY
ELSE IF domain is technology AND asks about specific version/config: VERIFY
ELSE IF domain is regulation AND asks about specific requirement: VERIFY
ELSE IF domain is science AND asks about guidelines/treatment: VERIFY
ELSE IF domain is news/events: VERIFY
ELSE IF domain is products AND asks about pricing/features: VERIFY
ELSE IF domain is mathematics/logic/history/philosophy: TRUST
ELSE IF domain is core programming concepts: TRUST
ELSE IF uncertain: VERIFY
DEFAULT: TRUST, but note uncertainty in response
```

### Step 3: Execute Verification

WHEN verification required, use appropriate tool:

| Domain | Tool | Query pattern |
|--------|------|---------------|
| Technology | Context7 | `resolve-library-id` → `query-docs` |
| Technology (no Context7 match) | WebSearch | `[tool] [version] [question] [current year]` |
| Regulations | WebSearch | `[regulation] current requirements [year] [jurisdiction]` |
| Regulations (authoritative) | WebFetch | Official government/regulatory body URL |
| Science/Medicine | WebSearch | `[topic] current guidelines [year]` |
| News/Events | WebSearch | `[topic] [current year]` |
| Products | WebSearch | `[product] pricing features [year]` |
| Products (official) | WebFetch | Official documentation URL |

MUST: Include current year in WebSearch queries for volatile topics
MUST: Prefer official/authoritative sources over general results
MUST NOT: Use WebSearch for stable knowledge (wastes time, no benefit)

### Step 4: Generate Response

#### 4.1 Response Format

```markdown
## Knowledge Assessment

**Verified (volatile):**
- [Item]: [Source] — [Key finding]
- [Item]: [Source] — [Key finding]

**Trusted training (stable):**
- [Item]: [Why stable]

**Confidence notes:**
- [Any remaining uncertainties]

---

[Main response content]

**For authoritative information:**
- [Official source 1]
- [Official source 2]
```

#### 4.2 Format Rules

MUST: Show knowledge assessment before main response
MUST: Cite sources for all verified information
MUST: Explain why trusted items are stable
MUST: List authoritative sources for high-stakes queries
SHOULD: Note any findings that differ from expected training knowledge

#### 4.3 Domain-Specific Requirements

WHEN domain is medical/health:
  MUST: Add disclaimer: "For medical decisions, consult a qualified healthcare provider. This information is for educational purposes only."
  MUST: Point to authoritative sources (FDA, NIH, medical associations)

WHEN domain is legal/regulatory:
  MUST: Add disclaimer: "For legal advice, consult a qualified attorney in your jurisdiction."
  MUST: Point to official regulatory body sources

WHEN domain is financial:
  MUST: Add disclaimer: "This is not financial advice. Consult a qualified financial advisor."

### Step 5: Validate Response

BEFORE returning, verify:

- [ ] Volatile knowledge was verified (not stated from memory)
- [ ] Stable knowledge was trusted (not unnecessarily verified)
- [ ] Sources cited for all verified claims
- [ ] Knowledge assessment section present
- [ ] Appropriate disclaimers for high-stakes domains
- [ ] Authoritative sources listed for high-stakes queries
- [ ] Uncertainties acknowledged

IF validation fails: Fix before returning.

## Definitions

VOLATILE KNOWLEDGE: Information that changes frequently — APIs, regulations, prices, current events, guidelines
STABLE KNOWLEDGE: Information that does not change — mathematical proofs, historical facts, scientific laws, core algorithms
VOLATILITY SIGNAL: Query terms indicating recency matters — "latest", "current", versions, dates
HIGH-STAKES QUERY: Medical, legal, or financial queries where incorrect information causes harm
KNOWLEDGE CUTOFF: Date after which Claude's training data does not include information

## Edge Cases

WHEN query mixes stable and volatile aspects:
  MUST: Verify volatile aspects only
  MUST: Trust stable aspects
  MUST: Clearly separate which is which in response

WHEN verification returns no results:
  MUST: State that current information was not found
  MUST: Provide training knowledge with explicit caveat: "Based on training data (may be outdated)"
  MUST NOT: Present training knowledge as current without caveat

WHEN query asks about something that does not exist yet:
  MUST: Verify existence before explaining
  IF does not exist: State clearly, do not hallucinate features

WHEN user asks about "best practices":
  MUST: Verify current practices (they evolve)
  MAY: Trust core principles underlying the practices

WHEN query is ambiguous about time frame:
  Ask: "Are you asking about current [topic] or the general concept?"
  OR: Provide both with clear labels

DEFAULT: When uncertain whether to verify, verify.

## Anti-Patterns

MUST NOT: Over-verify stable knowledge

```markdown
// WRONG: Verifying timeless knowledge
User: /verify explain quicksort
Claude: [searches for "quicksort algorithm 2025"]

// RIGHT: Trust stable CS fundamentals
User: /verify explain quicksort
Claude:
## Knowledge Assessment
**Trusted training (stable):** Quicksort algorithm (1959, unchanged)
**Verification needed:** None — sorting algorithms are foundational CS
```

MUST NOT: Under-verify volatile knowledge

```markdown
// WRONG: Stating versions from memory
User: /verify set up Next.js with Supabase
Claude: Use @supabase/auth-helpers...

// RIGHT: Verify fast-moving ecosystem
User: /verify set up Next.js with Supabase
Claude:
## Knowledge Assessment
**Verified:** Next.js v15, Supabase now uses @supabase/ssr (not auth-helpers)
```

MUST NOT: Present training knowledge as current for volatile topics

```markdown
// WRONG: False confidence
User: /verify what are GDPR cookie requirements
Claude: GDPR requires explicit consent for cookies. [no verification]

// RIGHT: Verify and cite
User: /verify what are GDPR cookie requirements
Claude:
## Knowledge Assessment
**Verified:** Checked 2025 EDPB guidance — cookie walls now explicitly non-compliant
**Source:** European Data Protection Board guidelines
```

MUST NOT: Skip disclaimers for high-stakes queries

```markdown
// WRONG: Medical info without disclaimer
User: /verify what medications interact with warfarin
Claude: [provides list without disclaimer]

// RIGHT: Verify, disclaim, cite authorities
User: /verify what medications interact with warfarin
Claude:
## Knowledge Assessment
**Verified:** Checked current drug interaction databases
**Important:** For medical decisions, consult a qualified healthcare provider.
**Authoritative sources:** FDA drug interaction checker, your pharmacist
```

## Tool Selection

| Situation | Tool | Rationale |
|-----------|------|-----------|
| Library/framework docs | Context7 | Most accurate, version-specific |
| Context7 returns no results | WebSearch | Fallback to general search |
| Official documentation needed | WebFetch | Direct source access |
| Current events/news | WebSearch | Broad coverage |
| Regulatory information | WebSearch → WebFetch | Find then verify at source |
| Stable knowledge | None | Trust training |

MUST: Try Context7 first for technology queries
MUST: Fall back to WebSearch if Context7 has no match
MUST NOT: Use WebFetch without knowing the URL (use WebSearch to find it first)
