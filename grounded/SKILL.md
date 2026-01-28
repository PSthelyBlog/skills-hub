# Grounded Skill

---
name: grounded
description: "Epistemic calibration — knowing when to verify vs. trust training knowledge"
version: 1.1.0
author: PSthelyBlog
tags: [knowledge, verification, accuracy, current-information, metacognition]
model: sonnet
examples:
  - "/grounded create a Next.js app with authentication"
  - "/grounded what are the current EU AI regulations"
  - "/grounded explain the latest treatment guidelines for Type 2 diabetes"
  - "/grounded how do I configure AWS Lambda with the newest runtime"
---

## Purpose

The `/grounded` skill teaches Claude epistemic calibration — the metacognitive ability to recognize when training knowledge might be outdated and verification is needed. Rather than defaulting to memory-first responses or checking everything (slow and unnecessary), Claude develops judgment about what it knows reliably vs. what requires current sources.

## The Problem

Claude's training has a knowledge cutoff. Without calibration:

- **Coding**: Outdated package versions, deprecated APIs
- **News/Events**: Missing post-cutoff developments
- **Regulations**: Superseded laws and compliance requirements
- **Science**: Outdated guidelines, new research findings
- **Products**: Changed features, pricing, availability
- **Standards**: Updated specifications, new versions
- **Best practices**: Evolved industry norms

## Core Philosophy

**Not everything ages equally.**

Some knowledge is timeless (mathematical proofs, historical events, logical principles). Some knowledge is volatile (framework APIs, regulations, current events). The skill is knowing which is which.

## The Knowledge Volatility Spectrum

```
STABLE ◄─────────────────────────────────────────────► VOLATILE

Math        Core CS       Established      Evolving        Current
Logic       Algorithms    Standards        Best Practices  Events
History     Language      Classic          Active          News
Philosophy  Fundamentals  Literature       Research        Releases
```

## The Decision Framework

### When to Verify (High Volatility)

| Domain | Signals | Examples |
|--------|---------|----------|
| **Current events** | Dates, "latest," "current," "now" | Election results, company news |
| **Technology** | Versions, setup, configuration | Framework APIs, cloud services |
| **Regulations** | Laws, compliance, requirements | GDPR, tax codes, licensing |
| **Science/Medicine** | Guidelines, recommendations | Treatment protocols, studies |
| **Products/Services** | Pricing, features, availability | SaaS tools, API capabilities |
| **Standards** | Specifications, protocols | W3C, ISO, RFCs, accessibility |
| **Statistics** | Numbers, data, metrics | Economic data, demographics |

### When to Trust Training (High Stability)

| Domain | Why Stable | Examples |
|--------|-----------|----------|
| **Mathematics** | Proofs don't expire | Calculus, linear algebra, statistics theory |
| **Core CS** | Fundamentals persist | Data structures, algorithms, complexity |
| **History** | Past events are fixed | Historical facts, established narratives |
| **Philosophy** | Ideas are timeless | Ethical frameworks, logical reasoning |
| **Language** | Syntax rarely changes | Grammar, core programming language features |
| **Literature** | Texts don't change | Book analysis, writing techniques |
| **Physics/Chemistry** | Laws are stable | Newtonian mechanics, periodic table |

### The Gray Zone (Use Judgment)

| Domain | Consider |
|--------|----------|
| **Established tech** | Stable but check if asking about latest features |
| **Scientific consensus** | Usually stable but active research areas shift |
| **Business practices** | Core principles stable, tactics evolve |
| **Legal principles** | Foundations stable, specific laws change |

## Instructions for Claude

When the user invokes `/grounded [query]`, follow this protocol:

### 1. Analyze the Query

Identify:
- **Domain**: What field does this touch?
- **Time sensitivity**: Is currency important?
- **Volatility signals**: "latest," "current," versions, dates, regulations
- **Stakes**: How much does accuracy matter here?

```
QUERY ANALYSIS:
- Domain(s): [technology / regulation / science / news / etc.]
- Time sensitivity: [high / medium / low]
- Volatility signals: [list any indicators]
- Accuracy stakes: [critical / important / casual]
```

### 2. Classify Knowledge Needs

For each aspect of the query:

**VERIFY** (use WebSearch, Context7, WebFetch):
- Post-cutoff events or developments
- Version-specific information
- Current regulations or guidelines
- Active pricing, features, availability
- Statistics or data points
- Anything with "latest," "current," "now"

**TRUST TRAINING**:
- Mathematical or logical principles
- Historical facts
- Core language/framework syntax
- Established scientific laws
- Timeless concepts and definitions
- Philosophical frameworks

**JUDGMENT CALL** (verify if uncertain):
- Established but evolving fields
- "Best practices" (stable core, shifting details)
- Scientific consensus (unless asking about recent research)

### 3. Execute Verification

When verification is needed:

**For technology:**
```
Context7: resolve-library-id → query-docs
WebSearch: "[tool] [version] [specific question] [current year]"
```

**For regulations/law:**
```
WebSearch: "[regulation] current requirements [year] [jurisdiction]"
WebFetch: Official government/regulatory body sites
```

**For science/medicine:**
```
WebSearch: "[topic] current guidelines [year]"
WebSearch: "[topic] latest research findings"
```

**For news/events:**
```
WebSearch: "[topic] [current year]"
WebSearch: "[company/person] latest news"
```

**For products/services:**
```
WebSearch: "[product] pricing features [year]"
WebFetch: Official product documentation
```

### 4. Show Your Reasoning

Always explain the epistemic assessment:

```
## Knowledge Assessment

**Verified (volatile/time-sensitive):**
- [What you looked up and why]
- [Key findings that differ from or confirm training]

**Trusted training (stable knowledge):**
- [What you used from training and why it's reliable]

**Confidence notes:**
- [Any remaining uncertainties]
- [Suggestions for authoritative sources if stakes are high]
```

### 5. Synthesize Response

Combine verified current information with stable training knowledge. Flag any uncertainties or areas where information might have changed since your search.

### 6. Offer Authoritative Sources

For high-stakes queries, always point to authoritative sources:

```
**For authoritative information:**
- [Official source 1]
- [Official source 2]
```

## The Five Questions

Before responding to any query, ask:

1. **Could this have changed since training?** → If yes, verify
2. **Is this a volatile domain?** → Technology, regulations, science updates → verify
3. **Are there volatility signals?** → "latest," "current," versions, dates → verify
4. **Is this timeless knowledge?** → Math, history, logic, core concepts → trust
5. **Am I uncertain?** → When in doubt, verify

## Domain-Specific Guidelines

### Technology & Software

**Verify:**
- Package versions and installation commands
- API syntax for fast-moving frameworks (React, Next.js, etc.)
- Cloud service configurations (AWS, GCP, Azure)
- Tool setup and configuration files
- Security best practices

**Trust:**
- Core language syntax (Python, JavaScript, etc.)
- Fundamental algorithms and data structures
- Database concepts (SQL basics, normalization)
- Design patterns and architectural principles
- Git commands and workflows

### Regulations & Compliance

**Verify:**
- Specific legal requirements and deadlines
- Tax rates and thresholds
- Privacy regulations (GDPR, CCPA specifics)
- Industry compliance standards
- Licensing requirements

**Trust:**
- Legal principles and frameworks
- Contract law basics
- General regulatory concepts
- Historical legal precedents

### Science & Medicine

**Verify:**
- Treatment guidelines and protocols
- Drug interactions and dosages
- Recent research findings
- Public health recommendations
- Clinical best practices

**Trust:**
- Basic biology, chemistry, physics
- Anatomical facts
- Established scientific laws
- Historical scientific discoveries

**Always add disclaimer:**
```
**Important:** For medical decisions, consult a qualified healthcare provider.
This information is for educational purposes only.
```

### News & Current Events

**Verify:**
- Anything that might have happened post-training
- Company status, leadership, acquisitions
- Political developments
- Economic indicators
- Ongoing situations

**Trust:**
- Historical context for events
- Geographical facts
- Biographical information (pre-cutoff)

### Products & Services

**Verify:**
- Pricing and plans
- Feature availability
- API capabilities and limits
- Service status and availability
- Comparison with competitors

**Trust:**
- General product categories
- Core functionality concepts
- Industry terminology

## Examples

### Example 1: Technology (Mostly Verify)

```
User: /grounded create a Next.js app with Supabase auth

Claude:
## Knowledge Assessment

**Verified (fast-moving ecosystem):**
- Next.js: v15 current, App Router default, React 19
- Supabase: Checked current auth patterns, @supabase/ssr package
- Integration: Current recommended approach uses middleware

**Trusted training (stable):**
- TypeScript syntax and patterns
- React component concepts
- General authentication flows

**Key finding:** Supabase auth setup has changed significantly—
now uses @supabase/ssr instead of @supabase/auth-helpers.

[Proceeds with current patterns...]
```

### Example 2: Regulation (Verify Specifics)

```
User: /grounded what are the GDPR requirements for cookie consent

Claude:
## Knowledge Assessment

**Verified (regulation specifics):**
- Current GDPR cookie requirements (2024 guidance)
- Recent enforcement trends and fines
- ePrivacy Directive interaction

**Trusted training (legal principles):**
- General concept of informed consent
- Data protection principles
- Privacy by design framework

**Key finding:** Enforcement has increased; "cookie walls"
now explicitly non-compliant per recent guidance.

**For authoritative information:**
- European Data Protection Board guidelines
- Your national Data Protection Authority

[Provides current requirements...]
```

### Example 3: Mixed Query (Partial Verify)

```
User: /grounded explain how neural networks learn and what the latest architectures are

Claude:
## Knowledge Assessment

**Trusted training (established science):**
- Backpropagation algorithm (1986, unchanged)
- Gradient descent fundamentals
- Basic neural network architectures (CNNs, RNNs)
- Mathematical foundations (linear algebra, calculus)

**Verified (active research area):**
- Latest architecture developments (2024-2025)
- Current state-of-the-art models
- Recent benchmark results

**Synthesis:** Core learning principles are stable and well-established.
For latest architectures, I checked current research...

[Explains fundamentals from training, latest developments from search...]
```

### Example 4: Stable Query (Trust Training)

```
User: /grounded explain the quicksort algorithm

Claude:
## Knowledge Assessment

**Trusted training (timeless computer science):**
- Quicksort algorithm (1959, unchanged)
- Time complexity analysis (established)
- Implementation patterns (stable)
- Comparison with other sorting algorithms (settled)

**Verification needed:** None. Sorting algorithms are
foundational computer science—they don't have versions
or updates.

[Explains directly from training knowledge...]
```

### Example 5: High-Stakes Query (Verify + Disclaim)

```
User: /grounded what medications interact with warfarin

Claude:
## Knowledge Assessment

**This is a high-stakes medical query.**

**Verified against current sources:**
- Checked current drug interaction databases
- Reviewed recent clinical guidelines

**Trusted training (established pharmacology):**
- General interaction mechanisms
- Drug metabolism principles

**Important:** Drug interactions are complex and individual.
This information is for educational purposes only.

**Always consult:**
- Your prescribing physician
- A licensed pharmacist
- FDA drug interaction resources

[Provides information with appropriate caveats...]
```

## Anti-Patterns to Avoid

**Don't over-verify:**
- Looking up basic syntax (`for` loops, array methods)
- Checking stable mathematical formulas
- Researching historical facts already well-established

**Don't under-verify:**
- Assuming framework APIs haven't changed
- Using remembered version numbers
- Stating current events from training
- Providing medical/legal specifics without checking

**Don't false-confidence:**
- Presenting outdated information confidently
- Ignoring volatility signals in the query
- Skipping verification because "I probably know this"

## Integration with Tools

**Context7** (preferred for technology):
```
resolve-library-id → query-docs
```

**WebSearch** (general current information):
```
Include current year in queries
Use authoritative domains when possible
```

**WebFetch** (specific authoritative sources):
```
Official documentation
Government/regulatory sites
Academic/medical databases
```

## Success Metrics

A grounded response:
- ✅ Correctly identifies what might be stale
- ✅ Verifies volatile information from current sources
- ✅ Trusts training for genuinely stable knowledge
- ✅ Shows reasoning about epistemic confidence
- ✅ Points to authoritative sources for high-stakes topics
- ✅ Acknowledges remaining uncertainties

## The Meta-Principle

**Calibrated confidence beats false certainty.**

It's better to say "let me check" for something that might have changed than to confidently provide outdated information. But it's also inefficient to check everything when much knowledge is genuinely stable.

The skill is knowing the difference.

---

**Remember:** Not all knowledge ages. Know what does.
