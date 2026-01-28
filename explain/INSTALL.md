# Installation Guide

Quick guide to installing the `/explain` skill in Claude Code.

## Prerequisites

- Claude Code CLI installed
- Basic familiarity with Claude Code skills system

## Installation Steps

### Option 1: Copy to Skills Directory (Recommended)

1. **Locate your Claude Code skills directory**:
   ```bash
   # Usually at:
   ~/.claude/skills/
   # or
   ~/Library/Application Support/Claude/skills/
   ```

2. **Copy the explain directory**:
   ```bash
   cp -r explain ~/.claude/skills/
   ```

3. **Verify installation**:
   ```bash
   # In Claude Code, type:
   /skills list
   # You should see "explain" in the list
   ```

4. **Test it**:
   ```bash
   /explain what does this do
   ```

### Option 2: Symbolic Link (For Development)

If you want to edit the skill and see changes immediately:

```bash
ln -s /path/to/multi-agent-brainstorm/explain ~/.claude/skills/explain
```

This creates a symlink, so any edits to the original files are immediately available.

### Option 3: Manual Installation

1. Create the skill directory:
   ```bash
   mkdir -p ~/.claude/skills/explain
   ```

2. Copy just the SKILL.md file:
   ```bash
   cp explain/SKILL.md ~/.claude/skills/explain/
   ```

3. (Optional) Copy documentation:
   ```bash
   cp explain/README.md ~/.claude/skills/explain/
   cp explain/EXAMPLES.md ~/.claude/skills/explain/
   ```

## Verification

After installation, verify the skill works:

```bash
# Quick test
/explain [any function name in your codebase]

# Deep test
/explain [complex concept] --deep
```

Expected behavior:
- Quick mode returns in <30 seconds
- Deep mode returns in <2 minutes
- Responses follow documented formats

## Troubleshooting

### Skill not found

**Problem**: `/explain` doesn't appear in `/skills list`

**Solutions**:
1. Check file location: `ls ~/.claude/skills/explain/SKILL.md`
2. Verify frontmatter syntax (YAML format)
3. Restart Claude Code
4. Check permissions: `chmod +r ~/.claude/skills/explain/SKILL.md`

### Skill runs but gives errors

**Problem**: Skill executes but Claude returns errors

**Solutions**:
1. Verify you're in a codebase (not an empty directory)
2. Check that Haiku model is available
3. Try a simpler query first: `/explain [filename]`

### Response is slow

**Problem**: Even quick mode takes >60 seconds

**Possible causes**:
1. Very large codebase (initial search is slow)
2. Model upgraded to Sonnet automatically (complex query)
3. Network latency

**Solutions**:
1. Be more specific in queries (target exact file/function)
2. Use quick mode for simple questions
3. Wait for Haiku model optimization

### --deep flag not working

**Problem**: Adding `--deep` doesn't change response format

**Verify**:
1. Flag syntax: `/explain [query] --deep` (flag at end)
2. Check SKILL.md instructions for --deep handling
3. Some simple queries may not need deep mode

## Customization

### Changing the Default Model

Edit `SKILL.md` frontmatter:

```yaml
---
name: explain
model: sonnet  # Change from 'haiku' to 'sonnet'
---
```

**Trade-off**: Sonnet is slower and more expensive, but provides richer analysis even in quick mode.

### Adding Custom Examples

Edit the frontmatter examples list:

```yaml
examples:
  - "/explain what does this function do"
  - "/explain authentication flow --deep"
  - "/your custom example here"
```

### Adjusting Response Format

Edit the "Structure Your Response" section in SKILL.md to customize:
- Number of bullet points in quick mode
- Sections included in deep mode
- Language/tone of responses

## Uninstallation

To remove the skill:

```bash
rm -rf ~/.claude/skills/explain
```

Or if using symlink:

```bash
unlink ~/.claude/skills/explain
```

## Getting Help

**Documentation**:
- `README.md` - Feature overview and usage
- `EXAMPLES.md` - Real-world scenarios
- `DESIGN.md` - Design rationale

**Common Issues**:
- Skill not found → Check installation path
- Slow responses → Use more specific queries
- Wrong format → Verify query syntax

**Testing**:
```bash
# Start simple
/explain formatDate

# Verify deep mode
/explain authentication --deep

# Test error handling
/explain nonexistent-function
# Should suggest alternatives, not just fail
```

## Updates

To update the skill:

1. **Backup current version** (if customized):
   ```bash
   cp -r ~/.claude/skills/explain ~/.claude/skills/explain.backup
   ```

2. **Copy new version**:
   ```bash
   cp -r explain ~/.claude/skills/
   ```

3. **Restart Claude Code** to load new version

4. **Test** to ensure no breaking changes

## Performance Tips

**For faster responses**:
- Be specific: "getUserById" not "user functions"
- Use quick mode first, escalate to --deep only if needed
- Work in smaller codebases when possible
- Target specific files: "/explain authentication in auth.js"

**For better quality**:
- Use --deep for unfamiliar subsystems
- Provide context: run from relevant directory
- Chain queries: understand parts before asking about whole

## Next Steps

After installation:

1. Read `EXAMPLES.md` for real-world usage patterns
2. Try a few quick queries to calibrate speed
3. Test --deep mode on a complex topic
4. Integrate into your workflow (explain before editing)

---

**Installation complete!** Start with `/explain [something]` and go from there.
