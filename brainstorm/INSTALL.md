# Installation Guide (v2.0)

Quick guide to installing the `/brainstorm` skill in Claude Code.

## Prerequisites

- Claude Code CLI installed
- Basic familiarity with Claude Code skills system

## Installation Steps

### Option 1: Copy to Skills Directory (Recommended)

1. **Locate your Claude Code skills directory**:
   ```bash
   # Usually at:
   ~/.claude/skills/
   ```

2. **Copy the brainstorm-skill directory**:
   ```bash
   cp -r brainstorm-skill ~/.claude/skills/brainstorm
   ```

3. **Verify installation**:
   ```bash
   ls ~/.claude/skills/brainstorm/SKILL.md
   ```

4. **Test it** (invoke DIRECTLY, not via skills-hub):
   ```bash
   /brainstorm should we use TypeScript or JavaScript
   ```

### Option 2: Symbolic Link (For Development)

If you want to edit the skill and see changes immediately:

```bash
ln -s /path/to/brainstorm-skill ~/.claude/skills/brainstorm
```

### Option 3: Manual Installation

1. Create the skill directory:
   ```bash
   mkdir -p ~/.claude/skills/brainstorm
   ```

2. Copy just the SKILL.md file (minimum required):
   ```bash
   cp brainstorm-skill/SKILL.md ~/.claude/skills/brainstorm/
   ```

3. (Optional) Copy documentation:
   ```bash
   cp brainstorm-skill/README.md ~/.claude/skills/brainstorm/
   cp brainstorm-skill/EXAMPLES.md ~/.claude/skills/brainstorm/
   ```

## Verification

After installation, verify the skill works correctly:

```bash
# Quick test - invoke DIRECTLY
/brainstorm monorepo vs polyrepo
```

**Expected behavior**:
- See real-time progress indicators
- 3 separate Task agents spawn in parallel
- Complete in ~30-45 seconds
- Structured output with Consensus, Trade-offs, Recommendation

**What to watch for**:
- If you see a "dialogue script" where agents talk to each other narratively, that's the theatrical fallback (wrong)
- Correct output shows collected results from 3 Tasks, then synthesis

## Critical: Direct Invocation Required

**v2.0 Discovery**: The Task tool is NOT available to subagents.

This means:
```bash
# CORRECT - invoke directly
/brainstorm should we use REST or GraphQL

# INCORRECT - delegation fails silently, produces theatrical output
/skills-hub brainstorm should we use REST or GraphQL
```

If using skills-hub, it should route brainstorm directly to the main session, not delegate to a subagent.

## Troubleshooting

### Problem: Skill not found

`/brainstorm` doesn't work or isn't recognized.

**Solutions**:
1. Check file location: `ls ~/.claude/skills/brainstorm/SKILL.md`
2. Verify frontmatter syntax (YAML format)
3. Restart Claude Code
4. Check permissions: `chmod +r ~/.claude/skills/brainstorm/SKILL.md`

### Problem: Getting theatrical/dialogue output

Instead of structured synthesis, you see agents "talking" to each other in a narrative format.

**Cause**: Skill was invoked via delegation (subagent), not directly.

**Solutions**:
1. Invoke `/brainstorm` directly, not via `/skills-hub`
2. If using a routing system, ensure brainstorm runs in main session
3. Verify you see "Task" tool invocations, not a single model writing a script

### Problem: Skill runs but is slow (>60 seconds)

**Possible causes**:
1. Network latency to API
2. Agents running sequentially instead of in parallel
3. API rate limiting

**Solutions**:
1. Check network connection
2. Verify parallel Task invocation (all 3 in single message)
3. Try a simpler topic first

### Problem: --agents flag not working

Custom agents don't seem to change perspectives.

**Verify**:
1. Flag syntax: `/brainstorm --agents a,b,c [topic]` (flag before topic)
2. Use valid persona names or descriptive terms
3. Separate with commas, no spaces

### Problem: Output missing sections

Synthesis is incomplete or malformed.

**Solutions**:
1. Ensure Round 2 completed before synthesis
2. Check that all 3 Task results were received
3. If network issues, retry

## Customization

### Adding Custom Personas

The skill supports 7 built-in personas. For custom perspectives:

```bash
/brainstorm --agents "cost-optimizer,platform-engineer,startup-founder" cloud architecture
```

Claude will interpret these names and adopt appropriate perspectives.

### Changing the Default Model

Edit `SKILL.md` frontmatter:

```yaml
---
name: brainstorm
model: sonnet  # Change from 'haiku' to 'sonnet' for agent calls
---
```

**Trade-off**: Sonnet is slower and more expensive, but provides richer responses.

### Modifying Output Format

Edit the "Synthesize Output" section in SKILL.md to customize:
- Number of sections
- Trade-off table format
- Next steps style

## Uninstallation

To remove the skill:

```bash
rm -rf ~/.claude/skills/brainstorm
```

Or if using symlink:

```bash
unlink ~/.claude/skills/brainstorm
```

## Migration from v1.0

If upgrading from v1.0:

1. **Backup current version** (if customized):
   ```bash
   cp -r ~/.claude/skills/brainstorm ~/.claude/skills/brainstorm.v1.backup
   ```

2. **Copy new version**:
   ```bash
   rm -rf ~/.claude/skills/brainstorm
   cp -r brainstorm-skill ~/.claude/skills/brainstorm
   ```

3. **Update invocation patterns**:
   - Change any `/skills-hub brainstorm` to `/brainstorm` direct invocation
   - Skills-hub delegation will produce theatrical (fake) output

4. **Test**:
   ```bash
   /brainstorm tabs vs spaces
   ```
   Verify you see 3 Task results being collected, not a dialogue script.

## Performance Tips

**For faster responses**:
- Keep topics focused (one decision at a time)
- Use default agents unless domain-specific perspectives matter
- Avoid very long topic descriptions

**For better quality**:
- Provide context: "for a 3-person startup" or "for enterprise scale"
- Use domain-specific agents: `--agents security,performance,ux`
- Be specific about constraints

## Getting Help

**Documentation**:
- `README.md` - Feature overview and usage
- `EXAMPLES.md` - Real-world scenarios
- `DESIGN.md` - Design rationale and architecture explanation

**Common Issues**:
- Theatrical output → Invoke directly, not via delegation
- Skill not found → Check installation path
- Slow responses → Normal, parallel agents take ~30 seconds

---

**Installation complete!**

Start with `/brainstorm [decision topic]` invoked directly (not via skills-hub delegation).
