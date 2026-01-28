# Suggest Skill for Claude Code

**Improve your codebase - from quick wins to comprehensive reviews.**

## What It Does

The `/suggest` skill analyzes your code and recommends actionable improvements with before/after diffs. It's the natural companion to `/explain` - where explain tells you what code does, suggest tells you what it could become.

## Quick Start

```bash
# Get suggestions for a file
/suggest src/auth.ts

# Focus on specific categories
/suggest --category security src/api/

# Apply suggestions interactively
/suggest --apply src/utils.ts

# Chain with explain
/explain src/payments.ts
/suggest                      # Uses same context
```

## Installation

1. Copy the `suggest` directory to your Claude Code skills folder
2. Restart Claude Code or reload skills
3. Type `/suggest` to start using it

## Core Features

### Actionable by Default
- Every suggestion includes before/after code
- One-sentence explanation of *why* it's better
- Ready to apply immediately

### Prioritized Output
- Limited to 5 suggestions by default (configurable)
- Ranked by impact: HIGH > MED > LOW
- High-impact, low-effort first

### Interactive Apply Mode
- Use `--apply` to step through suggestions
- Review each change before applying
- Skip what you don't want

## Example Session

```
You: /suggest src/database/queries.ts

Claude:
## Suggestions for src/database/queries.ts

### 1. [HIGH] Add error handling for database connection
**Category:** bugs
**Why:** Unhandled promise rejection will crash the process

**Before:**
```typescript
async function getUser(id: string) {
  const result = await db.query('SELECT * FROM users WHERE id = $1', [id]);
  return result.rows[0];
}
```

**After:**
```typescript
async function getUser(id: string) {
  try {
    const result = await db.query('SELECT * FROM users WHERE id = $1', [id]);
    return result.rows[0] ?? null;
  } catch (error) {
    logger.error('Failed to fetch user', { id, error });
    throw new DatabaseError('User lookup failed', { cause: error });
  }
}
```

---

### 2. [HIGH] Use parameterized query to prevent SQL injection
**Category:** security
**Why:** String concatenation in queries allows injection attacks

**Before:**
```typescript
const query = `SELECT * FROM orders WHERE status = '${status}'`;
```

**After:**
```typescript
const query = 'SELECT * FROM orders WHERE status = $1';
const result = await db.query(query, [status]);
```

---

**Summary:** 2 suggestions (2 high)
Run `/suggest --apply` to interactively apply these changes.

You: /suggest --apply

Claude:
Applying suggestion 1/2: Add error handling for database connection

[Shows before/after diff]

Apply this change? (y/n/skip all)

You: y

Claude: Applied. Moving to suggestion 2/2...
```

## Use Cases

### Code Review
Review code quality before merging
```bash
/suggest src/features/checkout/
/suggest --category security --depth thorough
```

### Quick Wins
Find easy improvements fast
```bash
/suggest --depth quick --limit 3 src/utils.ts
```

### Security Audit
Focus on vulnerabilities
```bash
/suggest --category security src/api/
```

### Performance Optimization
Find bottlenecks
```bash
/suggest --category performance --depth thorough
```

### Post-Explanation Improvement
Understand, then improve
```bash
/explain src/auth.ts --deep
/suggest --apply
```

## Design Philosophy

Built on three principles:

1. **Curated over comprehensive** (Pragmatist perspective)
   - 5 suggestions beats 50
   - Quality over quantity

2. **Show, don't tell** (Perfectionist perspective)
   - Before/after diffs for every suggestion
   - Structured severity levels

3. **Enable action** (User Advocate perspective)
   - Interactive apply mode
   - One-sentence explanations

## Flags Reference

| Flag | Description | Default |
|------|-------------|---------|
| `--category` | Filter: `bugs`, `security`, `performance`, `readability` | all |
| `--depth` | Analysis: `quick`, `standard`, `thorough` | standard |
| `--limit N` | Max suggestions (1-10) | 5 |
| `--apply` | Interactive apply mode | off |
| `--no-diffs` | Hide code blocks | show |
| `--json` | JSON output | off |

## Tips

### Get Better Suggestions
- Be specific: target files or functions, not directories
- Use category filters for focused analysis
- Run after `/explain` to reuse context

### When to Use Each Depth
- `quick` - Lint-style checks, obvious issues
- `standard` - Common issues + logical improvements (default)
- `thorough` - Architectural suggestions, cross-file patterns

### Chain with Other Skills
```bash
# Understand → Improve
/explain authentication middleware --deep
/suggest --apply

# Review workflow
/suggest --category bugs src/
/suggest --category security src/
/suggest --category performance src/
```

## Performance

- **Quick mode**: ~10-15 seconds
- **Standard mode**: ~20-30 seconds
- **Thorough mode**: ~45-60 seconds

Most queries should use standard mode. Use thorough for comprehensive reviews.

## Contributing

This skill emerged from a multi-agent brainstorm debate. See `DESIGN.md` for the full rationale and synthesis process.

Improvements welcome:
- Additional category patterns
- Better severity heuristics
- More apply mode options
- Language-specific suggestions

## License

Open source - use, modify, share freely.

## Credits

Synthesized by three agents:
- **Pragmatist**: Limited output, quick wins focus
- **Perfectionist**: Severity levels, structured format
- **User Advocate**: Before/after diffs, apply mode

Designed as the natural companion to `/explain`.

---

**Remember**: The best suggestion is the one you actually apply. Start with `/suggest`, end with better code.
