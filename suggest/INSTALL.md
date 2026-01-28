# Installation Guide

Quick guide to installing the `/suggest` skill in Claude Code.

## Prerequisites

- Claude Code CLI installed
- Basic familiarity with Claude Code skills system
- (Optional) `/explain` skill installed for best workflow

## Installation Steps

### Option 1: Copy to Skills Directory (Recommended)

1. **Locate your Claude Code skills directory**:
   ```bash
   # Usually at:
   ~/.claude/skills/
   # or
   ~/Library/Application Support/Claude/skills/
   ```

2. **Copy the suggest directory**:
   ```bash
   cp -r suggest ~/.claude/skills/
   ```

3. **Verify installation**:
   ```bash
   # In Claude Code, type:
   /skills list
   # You should see "suggest" in the list
   ```

4. **Test it**:
   ```bash
   /suggest src/index.ts
   ```

### Option 2: Symbolic Link (For Development)

If you want to edit the skill and see changes immediately:

```bash
ln -s /path/to/suggest ~/.claude/skills/suggest
```

This creates a symlink, so any edits to the original files are immediately available.

### Option 3: Manual Installation

1. Create the skill directory:
   ```bash
   mkdir -p ~/.claude/skills/suggest
   ```

2. Copy just the SKILL.md file:
   ```bash
   cp suggest/SKILL.md ~/.claude/skills/suggest/
   ```

3. (Optional) Copy documentation:
   ```bash
   cp suggest/README.md ~/.claude/skills/suggest/
   cp suggest/EXAMPLES.md ~/.claude/skills/suggest/
   ```

## Verification

After installation, verify the skill works:

```bash
# Basic test
/suggest [any file in your codebase]

# Category filter test
/suggest --category bugs src/

# Depth test
/suggest --depth quick src/utils.ts
```

Expected behavior:
- Standard mode returns in ~20-30 seconds
- Output includes before/after diffs
- Suggestions are ranked by severity

## Troubleshooting

### Skill not found

**Problem**: `/suggest` doesn't appear in `/skills list`

**Solutions**:
1. Check file location: `ls ~/.claude/skills/suggest/SKILL.md`
2. Verify frontmatter syntax (YAML format)
3. Restart Claude Code
4. Check permissions: `chmod +r ~/.claude/skills/suggest/SKILL.md`

### Skill runs but gives errors

**Problem**: Skill executes but Claude returns errors

**Solutions**:
1. Verify you're in a codebase (not an empty directory)
2. Specify a valid file path: `/suggest src/index.ts`
3. Try a simpler query first: `/suggest --depth quick src/`

### No suggestions found

**Problem**: "No significant improvements found"

**Possible causes**:
1. Code is actually well-written (congratulations!)
2. File is too small for meaningful suggestions
3. Depth is too shallow for the issues present

**Solutions**:
1. Try `--depth thorough` for deeper analysis
2. Target a larger file or directory
3. Remove category filter if using one

### Suggestions don't match codebase style

**Problem**: Suggested code doesn't follow project conventions

**Solutions**:
1. Suggestions follow common best practices
2. Adapt suggestions to your style when applying
3. Use `--apply` mode to review each change before accepting

### Apply mode not working

**Problem**: `--apply` flag doesn't prompt for changes

**Verify**:
1. Flag syntax: `/suggest --apply [target]` (flag before or after target)
2. File must be writable
3. Must have suggestions to apply (not "no improvements found")

## Customization

### Changing the Default Limit

Edit `SKILL.md` and modify the instructions to change the default from 5:

```markdown
4. Limit to N suggestions (default 5, respect `--limit` flag)
```

### Adding Custom Categories

The skill supports four categories. To suggest new patterns, edit the analysis section in SKILL.md:

```markdown
| **bugs** | Potential runtime errors, edge cases, null checks |
```

### Adjusting Severity Thresholds

Edit the severity assignment section to change what qualifies as HIGH/MED/LOW.

## Integration with /explain

For the best workflow, install both skills:

```bash
# Install both
cp -r explain-skill ~/.claude/skills/
cp -r suggest ~/.claude/skills/

# Use together
/explain src/auth.ts --deep
/suggest                      # Automatically uses same context
/suggest --apply              # Apply improvements
```

## Uninstallation

To remove the skill:

```bash
rm -rf ~/.claude/skills/suggest
```

Or if using symlink:

```bash
unlink ~/.claude/skills/suggest
```

## Getting Help

**Documentation**:
- `README.md` - Feature overview and usage
- `EXAMPLES.md` - Real-world scenarios
- `DESIGN.md` - Design rationale

**Common Issues**:
- Skill not found → Check installation path
- No suggestions → Try `--depth thorough`
- Wrong format → Verify flag syntax

**Testing**:
```bash
# Start simple
/suggest src/index.ts

# Verify categories work
/suggest --category security src/

# Test apply mode
/suggest --apply src/utils.ts
```

## Updates

To update the skill:

1. **Backup current version** (if customized):
   ```bash
   cp -r ~/.claude/skills/suggest ~/.claude/skills/suggest.backup
   ```

2. **Copy new version**:
   ```bash
   cp -r suggest ~/.claude/skills/
   ```

3. **Restart Claude Code** to load new version

4. **Test** to ensure no breaking changes

## Performance Tips

**For faster responses**:
- Use `--depth quick` for fast scans
- Target specific files, not entire directories
- Use `--limit 3` when you just need quick wins
- Chain after `/explain` to reuse context

**For better quality**:
- Use `--depth thorough` for comprehensive reviews
- Focus with `--category` when you know what you're looking for
- Review suggestions in `--apply` mode for learning

## Next Steps

After installation:

1. Read `EXAMPLES.md` for real-world usage patterns
2. Try a quick suggestion on a familiar file
3. Test `--apply` mode to see interactive workflow
4. Chain with `/explain` for the complete experience

---

**Installation complete!** Start with `/suggest [file]` and improve your code.
