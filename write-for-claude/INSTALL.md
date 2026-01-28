# Installation Guide

How to install and configure the `/write-for-claude` skill.

## Prerequisites

- Claude Code CLI installed
- Basic familiarity with Claude Code skills system

## Installation

### Option 1: Copy to Skills Directory (Recommended)

```bash
# Locate your Claude Code skills directory
# Usually at one of:
#   ~/.claude/skills/
#   ~/Library/Application Support/Claude/skills/

# Copy the skill
cp -r write-for-claude ~/.claude/skills/
```

### Option 2: Symbolic Link (For Development)

```bash
# Create symlink for live editing
ln -s /path/to/write-for-claude ~/.claude/skills/write-for-claude
```

### Option 3: Manual Installation

```bash
# Create directory
mkdir -p ~/.claude/skills/write-for-claude

# Copy only the core file
cp write-for-claude/SKILL.md ~/.claude/skills/write-for-claude/

# Optionally copy documentation
cp write-for-claude/README.md ~/.claude/skills/write-for-claude/
cp write-for-claude/EXAMPLES.md ~/.claude/skills/write-for-claude/
```

## Verification

After installation, verify the skill works:

```bash
# List available skills (should show write-for-claude)
/skills list

# Quick test
/write-for-claude convert "Try to keep code clean" to a directive

# Expected output includes MUST/SHOULD/MAY markers
```

## Usage Examples

### Generate a CLAUDE.md

```bash
/write-for-claude create a CLAUDE.md for this Python Django project
```

### Convert Existing Guidelines

```bash
/write-for-claude convert this style guide to unambiguous instructions
```

### Create Agent Definition

```bash
/write-for-claude define an agent for automated testing
```

### Write Skill Spec

```bash
/write-for-claude create a skill specification for formatting SQL queries
```

## Troubleshooting

### Skill Not Found

**Problem:** `/write-for-claude` doesn't appear in `/skills list`

**Solutions:**
1. Verify file location: `ls ~/.claude/skills/write-for-claude/SKILL.md`
2. Check YAML frontmatter syntax in SKILL.md
3. Restart Claude Code
4. Check file permissions: `chmod +r ~/.claude/skills/write-for-claude/SKILL.md`

### Output Not Following Format

**Problem:** Generated content doesn't use MUST/SHOULD/MAY markers

**Solutions:**
1. Be explicit in request: "using MUST/SHOULD/MAY priority markers"
2. Provide example of desired format in prompt
3. Check that SKILL.md instructions section is intact

### Slow Response

**Problem:** Skill takes longer than expected

**Causes:**
- Complex input requiring extensive transformation
- Large existing documentation being converted

**Solutions:**
1. Break large documents into sections
2. Process one section at a time
3. Be specific about which parts need transformation

## Customization

### Changing Priority Markers

Edit SKILL.md section "Use explicit priority markers":

```markdown
// Default markers
MUST: Non-negotiable requirements
SHOULD: Strong preferences
MAY: Optional behaviors

// Alternative (e.g., RFC 2119 style)
REQUIRED: Non-negotiable
RECOMMENDED: Strong preference
OPTIONAL: May be omitted
```

### Adding Custom Output Types

Add new type to "Identify the Output Type" table in SKILL.md:

```markdown
| **RUNBOOK.md** | Operational procedures | Steps, conditions, escalation |
```

Then add corresponding structure template in "Structure the Output" section.

### Adjusting Validation Checks

Modify the checklist in "Validate Before Output" section:

```markdown
// Add project-specific check
- [ ] All external URLs are to approved domains
```

## Integration Tips

### With CLAUDE.md

Use `/write-for-claude` when creating project instructions:

```bash
# Generate initial CLAUDE.md
/write-for-claude create CLAUDE.md for [project description]

# Refine existing CLAUDE.md
/write-for-claude review and improve this CLAUDE.md for ambiguity
```

### With Other Skills

Chain with `/explain` for understanding before writing:

```bash
# First understand
/explain how authentication works in this project --deep

# Then generate
/write-for-claude create CLAUDE.md section for authentication patterns
```

### For Team Standards

Generate team-wide guidelines once, then apply:

```bash
# Create standard template
/write-for-claude create a code review checklist template

# Apply to new project
/write-for-claude adapt this template for [new project]
```

## Uninstallation

```bash
# Remove skill
rm -rf ~/.claude/skills/write-for-claude

# Or if using symlink
unlink ~/.claude/skills/write-for-claude
```

## File Structure

```
write-for-claude/
├── SKILL.md      # Core specification (required)
├── README.md     # User documentation
├── EXAMPLES.md   # Before/after examples
├── INSTALL.md    # This file
└── LICENSE       # MIT license
```

## Updates

To update the skill:

```bash
# Backup customizations
cp ~/.claude/skills/write-for-claude/SKILL.md ~/.claude/skills/write-for-claude/SKILL.md.backup

# Copy new version
cp -r write-for-claude/* ~/.claude/skills/write-for-claude/

# Merge customizations back if needed
```

## Getting Help

**Documentation:**
- [README.md](README.md) - Feature overview
- [EXAMPLES.md](EXAMPLES.md) - Before/after transformations
- [SKILL.md](SKILL.md) - Full specification

**Common issues:**
- Skill not found → Check installation path
- Wrong output format → Be more explicit in request
- Missing sections → Verify SKILL.md is complete

---

**Installation complete!** Start with `/write-for-claude create a CLAUDE.md for this project`.
