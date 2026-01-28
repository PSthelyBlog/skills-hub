# Installation Guide

Quick guide to installing the `/skills-hub` skill in Claude Code.

## Prerequisites

- Claude Code CLI installed
- Basic familiarity with Claude Code skills system
- Other skills installed (explain, suggest, brainstorm, update, grounded, write-for-claude)

## Installation Steps

### Option 1: Copy to Skills Directory (Recommended)

1. **Locate your Claude Code skills directory**:
   ```bash
   # Usually at:
   ~/.claude/skills/
   # or
   ~/Library/Application Support/Claude/skills/
   ```

2. **Copy the skills-hub directory**:
   ```bash
   cp -r skills-hub ~/.claude/skills/
   ```

3. **Verify installation**:
   ```bash
   # In Claude Code, type:
   /skills list
   # You should see "skills-hub" in the list
   ```

4. **Test it**:
   ```bash
   /skills-hub explain what does this do
   ```

### Option 2: Symbolic Link (For Development)

If you want to edit the skill and see changes immediately:

```bash
ln -s /path/to/skills-hub ~/.claude/skills/skills-hub
```

This creates a symlink, so any edits to the original files are immediately available.

### Option 3: Manual Installation

1. Create the skill directory:
   ```bash
   mkdir -p ~/.claude/skills/skills-hub
   ```

2. Copy just the SKILL.md file:
   ```bash
   cp skills-hub/SKILL.md ~/.claude/skills/skills-hub/
   ```

3. (Optional) Copy documentation:
   ```bash
   cp skills-hub/README.md ~/.claude/skills/skills-hub/
   cp skills-hub/EXAMPLES.md ~/.claude/skills/skills-hub/
   ```

## Installing Companion Skills

The skills-hub routes to these skills, so they should also be installed:

```bash
# Install all companion skills
cp -r explain ~/.claude/skills/
cp -r suggest ~/.claude/skills/
cp -r brainstorm ~/.claude/skills/
cp -r update ~/.claude/skills/
cp -r grounded ~/.claude/skills/
cp -r write-for-claude ~/.claude/skills/
```

Or install only the ones you need — skills-hub will report missing skills gracefully.

## Verification

After installation, verify the skill works:

```bash
# Explicit routing
/skills-hub explain [any function in your codebase]

# Intent-based routing
/skills-hub what does the main function do

# Skill chaining
/skills-hub explain src/api.ts then suggest
```

Expected behavior:
- Explicit skill names route correctly
- Intent parsing selects appropriate skill
- Chained skills execute sequentially with context passing

## Troubleshooting

### Skill not found

**Problem**: `/skills-hub` doesn't appear in `/skills list`

**Solutions**:
1. Check file location: `ls ~/.claude/skills/skills-hub/SKILL.md`
2. Verify frontmatter syntax (YAML format)
3. Restart Claude Code
4. Check permissions: `chmod +r ~/.claude/skills/skills-hub/SKILL.md`

### Companion skill not found

**Problem**: Hub can't find a routed skill (e.g., explain)

**Solutions**:
1. Install the missing skill: `cp -r [skill] ~/.claude/skills/`
2. Check skill path in hub's routing logic
3. Verify companion skill's SKILL.md exists

### Routing to wrong skill

**Problem**: Hub selects unexpected skill for query

**Solutions**:
1. Use explicit skill name: `/skills-hub suggest [query]`
2. Check intent patterns in SKILL.md
3. Provide more context in your query

### Chaining not working

**Problem**: "then" keyword not triggering chain

**Verify**:
1. Syntax: `/skills-hub [skill1] [query] then [skill2] [query]`
2. Both skills are installed
3. Check SKILL.md for chaining instructions

### brainstorm fails in chain

**Problem**: Chaining with brainstorm produces errors

**Note**: brainstorm cannot be delegated (spawns its own Task agents). The hub handles this automatically, but if issues occur:
1. Execute brainstorm separately
2. Then run the follow-up skill with brainstorm's output as context

## Customization

### Adding a New Skill

1. Create skill directory: `~/.claude/skills/[new-skill]/`
2. Add SKILL.md with proper frontmatter
3. Update skills-hub SKILL.md:
   - Add to available skills table
   - Add intent patterns for routing
   - Add to skill reference section

### Changing Default Model

Edit `SKILL.md` frontmatter:

```yaml
---
name: skills-hub
model: opus  # Change from 'sonnet' to 'opus'
---
```

**Trade-off**: Opus provides better routing intelligence but is slower and more expensive.

### Modifying Intent Patterns

Edit the "Intent-Based Routing" section in SKILL.md to add or modify patterns:

```
| Intent Pattern | Route To |
|----------------|----------|
| "your new pattern" | your-skill |
```

## Uninstallation

To remove the skill:

```bash
rm -rf ~/.claude/skills/skills-hub
```

Or if using symlink:

```bash
unlink ~/.claude/skills/skills-hub
```

## Getting Help

**Documentation**:
- `README.md` - Feature overview and usage
- `EXAMPLES.md` - Real-world scenarios
- `DESIGN.md` - Design rationale

**Common Issues**:
- Skill not found → Check installation path
- Wrong routing → Use explicit skill names
- Chain fails → Check companion skills installed

**Testing**:
```bash
# Start simple
/skills-hub explain main

# Test chaining
/skills-hub explain src/api.ts then suggest

# Test intent routing
/skills-hub what does this function do
```

## Updates

To update the skill:

1. **Backup current version** (if customized):
   ```bash
   cp -r ~/.claude/skills/skills-hub ~/.claude/skills/skills-hub.backup
   ```

2. **Copy new version**:
   ```bash
   cp -r skills-hub ~/.claude/skills/
   ```

3. **Restart Claude Code** to load new version

4. **Test** to ensure no breaking changes

## Performance Tips

**For faster responses**:
- Use explicit skill names when you know which skill you need
- Avoid long chains (>3 skills) — break into separate requests
- Run independent analyses in parallel using "and"

**For better routing**:
- Be specific in your requests
- Use intent keywords that match skill purposes
- When in doubt, use explicit skill names

## Next Steps

After installation:

1. Read `EXAMPLES.md` for real-world usage patterns
2. Try basic routing with each skill
3. Test chaining between skills
4. Integrate into your workflow

---

**Installation complete!** Start with `/skills-hub [your query]` and let the hub route you to the right skill.
