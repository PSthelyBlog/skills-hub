# Installing Grounded Skill

## Prerequisites

- Claude Code CLI installed and working
- Internet access for verification capabilities
- Context7 MCP server (recommended for technology queries)

## Installation Methods

### Method 1: Project-Level (Recommended)

Add to your project's `.claude/skills/` directory:

```bash
# From your project root
mkdir -p .claude/skills
cp -r grounded .claude/skills/
```

The skill will be available when working in this project.

### Method 2: Global Installation

Add to your global Claude Code configuration:

```bash
# macOS/Linux
cp -r grounded ~/.claude/skills/

# Windows
copy grounded %USERPROFILE%\.claude\skills\
```

### Method 3: CLAUDE.md Integration

Reference the skill principles in your project's `CLAUDE.md`:

```markdown
## Epistemic Guidelines

Apply the grounded methodology for all responses:

Before answering, assess knowledge volatility:
- Verify: Technology versions, regulations, current events, statistics
- Trust training: Math, history, core concepts, stable language features
- Show reasoning: Explain what was verified and why

See: ./grounded/SKILL.md for full protocol
```

## Verifying Installation

After installation, test with:

```bash
# Should show "Knowledge Assessment" with verification
/grounded what's the latest version of React

# Should trust training without verification
/grounded explain how quicksort works
```

## Tool Dependencies

The skill uses these tools for verification:

### WebSearch (Required)
- Built into Claude Code
- No additional configuration needed
- Used for: news, regulations, general current information

### Context7 MCP (Recommended)
- Provides high-quality documentation queries
- Best for technology/framework verification

Add to your MCP configuration:
```json
{
  "mcpServers": {
    "context7": {
      "command": "npx",
      "args": ["-y", "@context7/mcp"]
    }
  }
}
```

### WebFetch (Built-in)
- Used for fetching specific authoritative pages
- No configuration needed

## Without Internet Access

If offline, Claude will:
1. Note that verification isn't possible
2. Provide training knowledge with caveats
3. Flag that information may be outdated
4. Recommend checking sources when online

## Customization

### Adjusting Domain Rules

Edit `SKILL.md` to customize verification triggers:

```markdown
### My Organization's Rules

**Always verify:**
- [Your industry's fast-moving regulations]
- [Technology stack you use]
- [Compliance requirements]

**Trust training for:**
- [Your established internal patterns]
- [Stable foundations in your domain]
```

### Adding Domain-Specific Sources

For specialized domains, add authoritative sources:

```markdown
**For [Your Domain] queries, check:**
- [Industry authority 1]
- [Official standards body]
- [Regulatory agency]
```

### Adjusting Disclaimer Thresholds

For high-stakes environments, increase disclaimers:

```markdown
**For any query touching [domain], always add:**
- Disclaimer about professional consultation
- Links to authoritative sources
- Verification date
```

## Integration with Other Skills

The grounded skill works alongside other skills:

```bash
# Use grounded methodology, then explain
/grounded explain how OAuth 2.0 works in modern SPAs

# Grounded code generation
/grounded create an API endpoint following REST best practices
```

## Troubleshooting

### "Context7 not available"
- Skill works without Context7 (uses WebSearch)
- For best tech documentation, configure Context7 MCP

### Verification seems slow
- Limit verification to truly volatile elements
- Use judgment—not everything needs checking
- Consider caching strategies for repeated queries

### Claude skips verification
- Ensure skill files are in correct location
- Check that skill is being invoked with `/grounded`
- Verify Claude Code has been restarted

### Too many disclaimers
- Adjust SKILL.md disclaimer thresholds
- Customize for your risk tolerance
- Consider domain-specific rules

## Uninstalling

Remove the skill directory:

```bash
# Project-level
rm -rf .claude/skills/grounded

# Global
rm -rf ~/.claude/skills/grounded
```

## Feedback & Improvements

The skill can be tuned based on:
- Your domain's volatility patterns
- Your accuracy requirements
- Your speed/thoroughness trade-offs

Adjust the heuristics in SKILL.md to match your needs.
