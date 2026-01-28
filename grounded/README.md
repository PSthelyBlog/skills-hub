# Grounded Skill for Claude Code

**Epistemic calibration — knowing when to verify vs. trust training knowledge.**

## The Problem

Claude has a training cutoff. Without calibration, it might:

- Recommend outdated package versions
- Miss recent regulatory changes
- Cite superseded medical guidelines
- Describe products with old features/pricing
- State post-cutoff events incorrectly

But verifying *everything* is slow and unnecessary — mathematical proofs don't expire, historical facts don't change, and core language syntax is stable.

## The Solution

The `/grounded` skill teaches Claude to distinguish:

| Verify | Trust Training |
|--------|----------------|
| Framework versions & APIs | Core language syntax |
| Current regulations | Legal principles |
| Recent research findings | Established science |
| Product pricing & features | Category definitions |
| Post-cutoff events | Historical facts |
| Active statistics | Mathematical proofs |

## Quick Start

```bash
# Technology (verify: fast-moving ecosystem)
/grounded create a Next.js app with Supabase auth

# Regulations (verify: laws change)
/grounded what are current GDPR cookie consent requirements

# Science (mixed: principles stable, guidelines evolve)
/grounded explain current treatment guidelines for hypertension

# Stable knowledge (trust training)
/grounded explain the quicksort algorithm

# Current events (verify: post-cutoff)
/grounded what's the current status of the EU AI Act
```

## How It Works

Before responding, Claude asks five questions:

1. **Could this have changed since training?**
2. **Is this a volatile domain?** (tech, regulations, medicine)
3. **Are there volatility signals?** ("latest," "current," versions)
4. **Is this timeless knowledge?** (math, history, core concepts)
5. **Am I uncertain?** (when in doubt, verify)

## Output Format

Claude shows its epistemic reasoning:

```
## Knowledge Assessment

**Verified (volatile/time-sensitive):**
- Next.js patterns (fast-moving, checked v15 docs)
- Current React version (19.0 stable)

**Trusted training (stable knowledge):**
- TypeScript syntax
- React component fundamentals

**Key findings:**
- App Router is now default
- New 'use server' directive for server actions
```

## Domain Coverage

### Technology
- Package versions and setup → **Verify**
- Core language syntax → **Trust**
- Cloud configurations → **Verify**
- Algorithms and data structures → **Trust**

### Regulations & Law
- Specific requirements and deadlines → **Verify**
- Legal principles and frameworks → **Trust**
- Recent enforcement trends → **Verify**

### Science & Medicine
- Treatment guidelines → **Verify + Disclaim**
- Established scientific laws → **Trust**
- Recent research findings → **Verify**

### Products & Services
- Pricing and features → **Verify**
- General product concepts → **Trust**
- API capabilities → **Verify**

### News & Events
- Anything potentially post-cutoff → **Verify**
- Historical context → **Trust**

## The Volatility Spectrum

```
STABLE ◄──────────────────────────────────────► VOLATILE

Math         Established     Evolving        Current
History      Standards       Best Practices  Events
Logic        Core Science    Active Research News
Philosophy   Language        Regulations     Releases
             Fundamentals    Product Details
```

## Examples

### High Verification (Technology Setup)
```
User: /grounded set up a Python project with FastAPI and SQLAlchemy

Claude verifies: FastAPI version, SQLAlchemy 2.0 patterns, Pydantic v2 syntax
Claude trusts: Python syntax, SQL concepts, REST principles
```

### Mixed (Scientific Query)
```
User: /grounded explain how mRNA vaccines work and current recommendations

Claude trusts: mRNA biology, immune system basics, protein synthesis
Claude verifies: Current vaccine schedules, recent efficacy data, guidelines
Claude disclaims: "Consult healthcare provider for personal medical advice"
```

### Low Verification (Stable Knowledge)
```
User: /grounded explain Big O notation and algorithm complexity

Claude trusts: Everything — this is foundational CS that doesn't change
Claude verifies: Nothing needed
```

## High-Stakes Queries

For medical, legal, and financial topics, Claude:

1. Still verifies against current sources
2. Adds appropriate disclaimers
3. Points to authoritative sources
4. Recommends professional consultation

```
**Important:** This information is for educational purposes.
For medical decisions, consult a qualified healthcare provider.

**Authoritative sources:**
- [Official source 1]
- [Official source 2]
```

## Installation

1. Copy `grounded/` to your Claude Code skills directory
2. Restart Claude Code
3. Use `/grounded` before queries where currency matters

See [INSTALL.md](INSTALL.md) for detailed instructions.

## When to Use `/grounded`

**Good candidates:**
- Setting up new projects or configurations
- Questions about current regulations or guidelines
- Topics that might have changed since 2024
- Anything where being wrong has consequences

**Not necessary:**
- Explaining core concepts and fundamentals
- Historical questions
- Mathematical or logical problems
- Following existing patterns in a codebase

## Philosophy

The skill embodies **calibrated confidence**:

- Confident about stable knowledge (don't waste time verifying proofs)
- Humble about volatile domains (check before asserting)
- Transparent about reasoning (show what was verified and why)

This mirrors expert human behavior — a doctor doesn't look up anatomy for every patient, but does check current drug interactions. A developer doesn't research `for` loops, but does check the latest framework migration guide.

## FAQ

**Q: Does this slow down responses?**
A: Slightly for volatile queries. But it's faster than debugging wrong information.

**Q: What if Claude can't verify something?**
A: It says so, provides best available information with caveats, and suggests authoritative sources.

**Q: Can I use this for any topic?**
A: Yes. The framework applies across domains — the same epistemic principles work for code, medicine, law, and current events.

## Contributing

Ideas welcome:
- Additional domain-specific heuristics
- Better volatility detection signals
- Integration with specialized verification sources
- Calibration for different accuracy requirements

## License

MIT — use, modify, share freely.

## Credits

Built on the principle that **not all knowledge ages equally** — and wisdom is knowing the difference.
