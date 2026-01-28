# Installation Guide

Quick guide to installing the `/update` skill in Claude Code.

## Prerequisites

- Claude Code CLI installed
- Context7 MCP plugin enabled (for documentation fetching)
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

2. **Copy the update directory**:
   ```bash
   cp -r update ~/.claude/skills/update
   ```

3. **Verify installation**:
   ```bash
   # In Claude Code, type:
   /skills list
   # You should see "update" in the list
   ```

4. **Test it**:
   ```bash
   /update --help
   ```

### Option 2: Symbolic Link (For Development)

If you want to edit the skill and see changes immediately:

```bash
ln -s /path/to/update ~/.claude/skills/update
```

This creates a symlink, so any edits to the original files are immediately available.

### Option 3: Manual Installation

1. Create the skill directory:
   ```bash
   mkdir -p ~/.claude/skills/update
   ```

2. Copy just the SKILL.md file:
   ```bash
   cp update/SKILL.md ~/.claude/skills/update/
   ```

3. (Optional) Copy documentation:
   ```bash
   cp update/README.md ~/.claude/skills/update/
   cp update/EXAMPLES.md ~/.claude/skills/update/
   ```

## Context7 Setup

The `/update` skill requires Context7 for fetching fresh documentation.

### Verify Context7 is Available

```bash
# In Claude Code, try:
/mcp status
# Should show context7 plugin as active
```

### If Context7 is Not Available

1. Check your MCP configuration
2. Ensure the Context7 plugin is installed
3. Restart Claude Code

Without Context7, the skill will show an error:
```
Documentation service unavailable. Cannot proceed without fresh documentation.
```

## Verification

After installation, verify the skill works:

### Basic Test

```bash
# Create a test file
echo "import React from 'react';" > /tmp/test-update.tsx

# Run update with dry-run
/update /tmp/test-update.tsx --dry-run
```

Expected behavior:
- Detects React as a library
- Prompts for confirmation
- Fetches documentation
- Shows proposed changes (if any)
- Does not modify file (dry-run mode)

### Full Test

```bash
# Test with an actual project file
/update src/components/Example.tsx --dry-run --verbose
```

This should:
- Detect libraries from imports
- Fetch docs from Context7
- Show detailed proposed changes
- Display risk levels

## Troubleshooting

### Skill Not Found

**Problem**: `/update` doesn't appear in `/skills list`

**Solutions**:
1. Check file location: `ls ~/.claude/skills/update/SKILL.md`
2. Verify frontmatter syntax (YAML format)
3. Restart Claude Code
4. Check permissions: `chmod +r ~/.claude/skills/update/SKILL.md`

### Context7 Errors

**Problem**: "Documentation service unavailable"

**Solutions**:
1. Check MCP status: `/mcp status`
2. Verify Context7 plugin is configured
3. Test Context7 directly:
   ```
   # Try fetching docs for a known library
   mcp context7 resolve-library-id --libraryName react
   ```
4. Restart Claude Code to reinitialize MCP

### Library Not Found

**Problem**: "Library [name] not found in Context7"

**Solutions**:
1. Use exact npm package name with `--lib`
2. Try common alternative names
3. Some niche libraries may not be indexed

### Changes Look Wrong

**Problem**: Proposed changes seem incorrect

**Solutions**:
1. Always use `--dry-run` first
2. Check the documentation reference in each change
3. Reject changes you disagree with
4. File an issue for incorrect suggestions

### Performance Issues

**Problem**: Skill takes too long

**Possible causes**:
1. Large files with many imports
2. Network latency to Context7
3. Many libraries to detect

**Solutions**:
1. Use `--lib [name]` to target specific library
2. Use `--scope function` to limit changes
3. Target specific files rather than directories

## Customization

### Changing Default Model

Edit `SKILL.md` frontmatter:

```yaml
---
name: update
model: opus  # Change from 'sonnet' to 'opus' for more complex analysis
---
```

**Note**: This skill uses Sonnet by default because code modifications require careful reasoning. Downgrading to Haiku is not recommended.

### Adjusting Default Behavior

The skill defaults are:
- `--scope file` (update entire file)
- `--max-risk high` (show all changes)
- Backup enabled
- Confirmation required

These cannot be changed without modifying `SKILL.md`, but you can always override with flags:
```bash
/update [file] --scope function --max-risk low --yes --no-backup
```

### Adding Custom Library Aliases

If a library is commonly known by a different name, you can suggest improvements to the skill's detection logic in `SKILL.md`.

## Uninstallation

To remove the skill:

```bash
rm -rf ~/.claude/skills/update
```

Or if using symlink:

```bash
unlink ~/.claude/skills/update
```

## File Structure

After installation, your skills directory should look like:

```
~/.claude/skills/
└── update/
    ├── SKILL.md          # Required - skill specification
    ├── README.md         # Optional - user documentation
    ├── EXAMPLES.md       # Optional - usage scenarios
    ├── DESIGN.md         # Optional - design rationale
    └── INSTALL.md        # Optional - this file
```

Only `SKILL.md` is required. The other files provide documentation.

## Updates

To update the skill:

1. **Backup current version** (if customized):
   ```bash
   cp -r ~/.claude/skills/update ~/.claude/skills/update.backup
   ```

2. **Copy new version**:
   ```bash
   cp -r update/* ~/.claude/skills/update/
   ```

3. **Restart Claude Code** to load new version

4. **Test** to ensure no breaking changes

## Integration with Workflows

### Before Pull Requests

```bash
# Check for updates before submitting PR
/update src/ --lib [library] --dry-run
```

### CI Pipeline

```bash
# Apply only safe changes in CI
/update src/ --yes --max-risk low
```

### Post-Dependency Update

After running `npm update`:
```bash
# Modernize code to match new versions
/update src/ --scope all
```

## Getting Help

**Documentation**:
- `README.md` - Feature overview and usage
- `EXAMPLES.md` - Real-world scenarios
- `DESIGN.md` - Design rationale

**Testing**:
```bash
# Start simple
/update [small-file] --dry-run

# Verify with verbose
/update [file] --dry-run --verbose

# Test error handling
/update nonexistent-file.ts
# Should show helpful error
```

## Next Steps

After installation:

1. Read `EXAMPLES.md` for real-world usage patterns
2. Try `--dry-run` on a few files to see what would change
3. Start with low-risk changes before trusting high-risk ones
4. Review backups and audit logs to understand the safety net

---

**Installation complete!** Start with `/update [file] --dry-run` to explore safely.
