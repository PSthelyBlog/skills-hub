# Explain Skill for Claude Code

**Understand your codebase - from quick answers to deep dives.**

## What It Does

The `/explain` skill turns Claude Code into an intelligent teaching assistant that helps you understand code, patterns, and architecture. Instead of just executing commands, it explains WHY things work the way they do.

## Quick Start

```bash
# Simple questions get fast answers
/explain what does getUserById do

# Add --deep for comprehensive analysis
/explain authentication flow --deep

# Debug with context
/explain why is the build failing

# Understand relationships
/explain dependency between users and posts
```

## Installation

1. Copy the `explain` directory to your Claude Code skills folder
2. Restart Claude Code or reload skills
3. Type `/explain` to start using it

## Core Features

### Fast by Default
- Uses Haiku model for speed (5x faster than Sonnet)
- Concise 3-point format
- Answers in seconds, not minutes

### Deep When Needed
- Add `--deep` flag for comprehensive analysis
- Automatic upgrade to Sonnet for complex topics
- Covers dependencies, design choices, gotchas

### Teaching-Oriented
- Plain language explanations
- Uses YOUR code as examples
- Progressive disclosure - learn as much as you want
- Builds understanding, not dependency

## Example Session

```
You: /explain processPayment

Claude:
## processPayment

**What it does**: Handles credit card charges via Stripe API

**Key points**:
- Validates amount and currency before charging
- Returns transaction ID on success, null on failure
- Automatically retries once on network errors

**Why it matters**: Safe to call without try-catch, but check for null returns.

---
Need more? Add --deep for retry logic and error handling details.

You: /explain processPayment --deep

Claude:
## processPayment

**Overview**: Handles credit card charges through Stripe API with built-in
retry logic, idempotency key support, and comprehensive error handling.
Integrates with internal transaction logging and customer notification systems.

**How it works**:
1. Validates amount (>0, <=999999) and currency (USD/EUR/GBP)
2. Generates idempotency key from order ID + timestamp
3. Calls Stripe API with payment method token
4. On network error: waits 2s, retries once
5. On success: logs transaction, queues confirmation email
6. On failure: logs error, returns null (doesn't throw)

**Dependencies**:
- Uses: stripe-node, transaction-logger, email-queue
- Used by: checkout-controller, subscription-manager

**Design choices**:
- Idempotency keys prevent duplicate charges on retry
- Returns null vs throwing to avoid cascading failures
- Single retry balances reliability vs latency
- Async email queuing prevents blocking payment flow

**Common patterns**:
Follows the "defensive external API" pattern used throughout payment flows.
Similar retry logic in refundPayment and updateSubscription.

**Gotchas**:
- Null return could mean network failure OR insufficient funds - check logs
- Idempotency keys expire after 24h (Stripe limit)
- Test mode uses different API keys - check environment

**Related**:
- `services/stripe-client.js` - API wrapper
- `models/transaction.js` - Transaction logging
- `tests/payment.test.js` - Test coverage including edge cases
```

## Use Cases

### For Learning
Understand unfamiliar code before modifying it
```bash
/explain how our caching works --deep
/explain the database migration system
```

### For Debugging
Get context when things break
```bash
/explain why this test is failing
/explain what's calling this deprecated function
```

### For Code Review
Understand impact of changes
```bash
/explain what this PR changes --deep
/explain potential side effects of this refactor
```

### For Onboarding
Learn the codebase faster
```bash
/explain project structure
/explain our API conventions --deep
```

## Design Philosophy

Built on three principles:

1. **Start simple, go deep on request** (Pragmatist perspective)
   - Default mode is fast and concise
   - Optional depth for those who need it

2. **Comprehensive when it matters** (Completionist perspective)
   - Deep mode covers dependencies, design choices, gotchas
   - Structured analysis, not stream-of-consciousness

3. **Always teach, never just execute** (User Advocate perspective)
   - Plain language before technical terms
   - Shows WHY, not just WHAT
   - Builds mental models

## Tips

### Get Better Answers
- Be specific: "getUserById" > "user functions"
- Use context: run from relevant directory
- Try quick first, then add --deep if needed

### Chain with Other Tools
```bash
# Explain before editing
/explain auth middleware
# [understand it]
# Now modify with confidence

# Explain after errors
# [test fails]
/explain why this test failed
# [get context to fix]
```

### When to Use --deep
- Unfamiliar architecture
- Before major refactoring
- Complex debugging
- Learning new subsystems
- Code review of significant changes

### When NOT to Use --deep
- Simple "what does this do"
- Quick error lookups
- Familiar code patterns
- Time-sensitive answers

## Performance

- **Quick mode**: ~5-10 seconds (Haiku)
- **Deep mode**: ~30-60 seconds (Sonnet)
- **Cost**: ~10x cheaper for quick mode

80% of queries should use quick mode. Deep mode is there when you genuinely need comprehensive analysis.

## Contributing

This skill emerged from a multi-agent brainstorm debate. See `DESIGN.md` for the full rationale and synthesis process.

Improvements welcome:
- Better error handling suggestions
- More context awareness patterns
- Additional response formats
- Performance optimizations

## License

Open source - use, modify, share freely.

## Credits

Synthesized by three agents:
- **Pragmatist**: Simplicity and speed
- **Completionist**: Depth and thoroughness
- **User Advocate**: Teaching and progressive disclosure

Orchestrated via Claude Code Task tool multi-agent brainstorm pattern.
