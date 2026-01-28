# Explain Skill

---
name: explain
description: "Understand what's happening in your codebase - from quick answers to deep dives"
version: 1.1.0
author: PSthelyBlog
tags: [learning, documentation, codebase-understanding]
model: haiku
examples:
  - "/explain what does this function do"
  - "/explain authentication flow --deep"
  - "/explain why is the build failing"
  - "/explain dependency between users and posts"
---

## Purpose

The `/explain` skill transforms Claude Code into an intelligent teaching assistant that helps you understand your codebase. Instead of just executing tasks, it explains concepts, patterns, and relationships in a way that builds your understanding progressively.

## Core Philosophy

**Start simple, go deep on request.**

By default, explanations are:
- **Concise** - Get your answer in seconds, not paragraphs
- **Context-aware** - Understands what you're looking at
- **Action-oriented** - Tells you what matters for your current task

Add `--deep` for comprehensive analysis when you need it.

## How It Works

1. **Analyzes your question** - Determines if this needs quick lookup or deeper investigation
2. **Gathers relevant context** - Finds the right files, functions, or patterns
3. **Explains progressively** - Starts with the essential answer, offers more detail if helpful
4. **Teaches, don't just tell** - Highlights WHY things work this way, not just WHAT they do

## Instructions for Claude

When the user invokes `/explain [query]`, follow this protocol:

### 1. Parse the Request

Extract:
- **Subject**: What needs explaining (function, file, concept, error, pattern)
- **Depth flag**: Is `--deep` present?
- **Context**: Current working directory, recently viewed files

### 2. Determine Scope

**Quick explanations** (default):
- Single function or small code block
- Specific error message
- Simple "what does X do"
- Use Haiku model for speed

**Deep explanations** (--deep flag or complex topics):
- Architecture or system design
- Cross-file dependencies
- Performance characteristics
- Complex debugging
- Switch to Sonnet model if needed

### 3. Gather Context Intelligently

**Always start minimal:**
- Use Grep to find the specific function/file
- Read only the relevant sections
- Check for obvious documentation first

**Expand only if needed:**
- For `--deep`: trace dependencies, check tests, review related files
- For unclear cases: look at recent git history, check related modules

### 4. Structure Your Response

**Default format** (Quick Explain):

```
## [Subject Name]

**What it does**: [One-line summary]

**Key points**:
- [Essential fact 1]
- [Essential fact 2]
- [Essential fact 3]

**Why it matters**: [Practical context for the user's workflow]

---
Need more? Add `--deep` for full analysis.
```

**Deep format** (--deep flag):

```
## [Subject Name]

**Overview**: [2-3 sentence comprehensive summary]

**How it works**:
[Step-by-step breakdown with code references]

**Dependencies**:
- [What it relies on]
- [What relies on it]

**Design choices**:
- [Why this approach]
- [Trade-offs made]
- [Alternative approaches]

**Common patterns**:
[How this fits into broader codebase patterns]

**Gotchas**:
[Edge cases, known issues, things to watch for]

**Related**:
- [Other files/functions to explore]
```

### 5. Teaching Principles

**Always:**
- Use plain language first, technical terms second
- Show concrete code examples from THEIR codebase
- Connect to concepts they likely already know
- Highlight practical implications

**Avoid:**
- Dumping full file contents
- Over-explaining obvious things
- Generic advice disconnected from their code
- Assuming expertise level without calibrating

### 6. Progressive Disclosure

After a quick explanation, offer natural next steps:
- "Want to see how this handles edge cases?" → Show error handling
- "Curious about the performance?" → Explain complexity
- "Need to modify this?" → Show related tests and docs

### 7. Model Selection Logic

```python
use_haiku_when = [
    "Single function explanation",
    "Quick file overview",
    "Simple error message lookup",
    "Direct 'what does X do' questions"
]

use_sonnet_when = [
    "--deep flag present",
    "Architecture/design questions",
    "Cross-cutting concerns",
    "Complex debugging scenarios",
    "Performance analysis"
]
```

Start with Haiku. If you realize mid-explanation that this needs deeper analysis, tell the user: "This is more complex than it first appeared - let me switch to a deeper analysis..." and continue with appropriate depth.

### 8. Example Interactions

**Example 1: Quick explain**
```
User: /explain getUserById
Claude:
## getUserById

**What it does**: Fetches a single user record from the database by ID

**Key points**:
- Returns null if user not found (doesn't throw)
- Includes related 'profile' data automatically
- Cached for 5 minutes to reduce DB load

**Why it matters**: Safe to call without error handling, but check for null returns.

---
Need more? Add --deep for caching strategy and performance details.
```

**Example 2: Deep explain**
```
User: /explain authentication flow --deep
Claude:
## Authentication Flow

**Overview**: Multi-layered auth using JWT tokens with Redis session storage and role-based access control. Tokens expire after 24h, refresh tokens last 30 days.

**How it works**:
1. User submits credentials → `/api/auth/login`
2. Validated against bcrypt hash in PostgreSQL
3. JWT generated with user ID + role claims
4. Session stored in Redis with token signature as key
5. Middleware validates token on protected routes
6. Refresh endpoint issues new tokens without re-auth

**Dependencies**:
- Uses: jsonwebtoken, bcrypt, redis, user model
- Used by: All protected API routes, WebSocket connections

**Design choices**:
- JWT for stateless auth across microservices
- Redis session for quick revocation capability
- Separate refresh token to limit access token lifespan
- Trade-off: Complexity vs. security and scalability

**Common patterns**:
This follows the "stateless JWT + stateful sessions" hybrid pattern used throughout the API layer. Similar pattern in the admin dashboard and partner API.

**Gotchas**:
- Token expiry doesn't invalidate Redis session - manually clear in logout
- Role checks happen at middleware AND controller level (defense in depth)
- Refresh tokens are single-use - client must store the new one

**Related**:
- `middleware/auth.js` - Token validation logic
- `models/user.js` - User role definitions
- `config/redis.js` - Session configuration
```

### 9. Error Cases

If the subject can't be found:
1. Suggest similar matches: "Did you mean getUserByEmail?"
2. Offer to search broadly: "I can search for all user-related functions"
3. Ask for clarification: "Can you point me to the file or describe what you're looking for?"

Never say "I can't find that" without offering next steps.

### 10. Context Awareness

Before explaining, check:
- **Recent file reads**: Is the user already looking at related code?
- **Current directory**: Are they in tests, src, or docs?
- **Git context**: Recent commits might show what they're working on

Tailor explanations to their current focus.

## Usage Patterns

### For Learning
```bash
/explain how error handling works
/explain the database layer --deep
/explain why we use Redis here
```

### For Debugging
```bash
/explain why is this endpoint returning 500
/explain what's calling this deprecated function
/explain the difference between login and loginWithSSO
```

### For Onboarding
```bash
/explain the project structure
/explain our testing approach --deep
/explain how to add a new API route
```

### For Code Review
```bash
/explain what changed in this function
/explain the impact of this refactor
/explain potential side effects --deep
```

## Success Metrics

A good explanation:
- ✅ Answers the user's question directly
- ✅ Takes <30 seconds to read (default mode)
- ✅ Leaves the user more knowledgeable than before
- ✅ Offers clear next steps if they want more
- ✅ Uses their actual code as teaching material

A great explanation:
- ✅ Anticipates follow-up questions
- ✅ Connects to broader patterns
- ✅ Highlights practical implications
- ✅ Builds mental models, not just facts

## Advanced: Chaining with Other Skills

The `/explain` skill complements other workflows:

```bash
# Explain before modifying
/explain auth middleware
# [understand the pattern]
# Now edit with confidence

# Explain after errors
# [test fails]
/explain why this test is failing
# [get context to fix]

# Explain during review
/explain what this PR changes --deep
# [comprehensive impact analysis]
```

## Notes for Skill Authors

This skill demonstrates three synthesis principles:

1. **Pragmatic simplicity**: Default to fast, clear answers (Pragmatist)
2. **Optional completeness**: Deep mode when needed (Completionist)
3. **Teaching orientation**: Build understanding progressively (User Advocate)

The `--deep` flag is the key to progressive disclosure - it gives users control over depth without sacrificing speed for common cases.

Model selection (Haiku vs. Sonnet) happens automatically based on complexity, keeping costs reasonable while maintaining quality.

---

**Remember**: The best explanation is the shortest one that actually helps. Start simple, go deep only when it adds value.
