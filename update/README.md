# Update Skill for Claude Code

**Modernize code using fresh documentation to overcome AI training cutoffs.**

## What It Does

The `/update` skill fetches current library documentation at runtime, ensuring your code updates are based on the latest APIs—not outdated training data. When you need to migrate from React Query v3 to v5, or upgrade Express middleware, this skill knows what changed.

## Quick Start

```bash
# Auto-detect libraries from imports
/update src/components/UserProfile.tsx

# Specify a library explicitly
/update src/api/users.ts --lib express

# Preview changes without applying
/update src/hooks/useAuth.ts --dry-run

# Update all files using a library
/update src/ --lib zod --scope all
```

## Installation

1. Copy the `update` directory to your Claude Code skills folder
2. Restart Claude Code or reload skills
3. Type `/update [file]` to start using it

## Core Features

### Fresh Documentation
- Fetches current docs via Context7
- Knows about recent breaking changes
- References actual migration guides

### Safe by Default
- Always shows diff before applying
- Creates backup files automatically
- Per-change approval workflow
- Audit logging for every operation

### Smart Detection
- Auto-detects libraries from imports
- Reads versions from package.json
- Flags breaking changes with warnings

### Granular Control
- Filter by risk level (low/medium/high)
- Accept or reject individual changes
- Scope to function, file, or entire project

## Example Session

```
You: /update src/hooks/useData.ts

Claude:
🔍 Scanning imports...
📦 Detected: react-query@3.39.0

   Update this library? [Y/n]

You: Y

📚 Fetching current documentation...

## Update Opportunities

### react-query → @tanstack/react-query (v5)
⚠️ BREAKING: Package renamed, API changes

📊 Summary:
- 🟢 2 import updates
- 🟡 3 API signature changes
- 🔴 0 behavioral changes

---

Change 1 of 5: Import path update (🟢 Low)

```diff
- import { useQuery } from 'react-query'
+ import { useQuery } from '@tanstack/react-query'
```

[Accept] [Reject] [Accept All] [Show More]

You: Accept All

## Update Complete

✅ Applied 5 changes to src/hooks/useData.ts
📁 Backup: .update-backups/src_hooks_useData.ts.1706140800.bak

Next steps:
- Run your tests to verify changes
- Delete backup after confirming
```

## Use Cases

### Library Migration
Update to the latest version of a library with confidence
```bash
/update src/api/ --lib express --scope all
```

### Breaking Change Discovery
Preview what would change without modifying files
```bash
/update src/components/ --dry-run --verbose
```

### Safe Production Updates
Apply only low-risk changes automatically
```bash
/update src/ --max-risk low --yes
```

### Targeted Updates
Update just one function at a time
```bash
/update src/utils/auth.ts --scope function
```

## Arguments Reference

| Argument | Type | Default | Description |
|----------|------|---------|-------------|
| `target` | string | required | File or directory to update |
| `--lib` | string | auto-detect | Specific library to update |
| `--scope` | enum | `file` | `function`, `file`, or `all` |
| `--dry-run` | boolean | false | Preview without applying |
| `--verbose` | boolean | false | Show detailed output |
| `--yes` | boolean | false | Skip confirmation prompts |
| `--max-risk` | enum | `high` | Maximum risk level: `low`, `medium`, `high` |
| `--no-backup` | boolean | false | Skip backup creation |

## Risk Levels

Each proposed change is classified by potential impact:

| Level | Icon | Description | Example |
|-------|------|-------------|---------|
| Low | 🟢 | Safe renames, import changes | Package path updates |
| Medium | 🟡 | API signature changes | Function argument restructuring |
| High | 🔴 | Behavioral changes | Caching or timing differences |

Breaking changes are marked with ⚠️ regardless of risk level.

## Safety Features

### Backup Before Modify
Every change creates a timestamped backup:
```
.update-backups/src_components_UserProfile.tsx.1706140800.bak
```

### Diff Preview
See exactly what changes before they're applied:
- Side-by-side comparison
- Risk level for each change
- Documentation references

### Per-Change Approval
Accept or reject individual changes:
```
Change 1 of 5: [Accept] [Reject] [Accept All] [Reject All]
```

### Audit Logging
All operations logged to `.update-audit/`:
- What was detected
- What docs were fetched
- What was proposed/applied/rejected

## Design Philosophy

Built on three principles from multi-agent synthesis:

1. **Safety First** (Perfectionist perspective)
   - Never apply without preview
   - Always create backups
   - Verify against documentation

2. **Developer Experience** (User Advocate perspective)
   - Auto-detect libraries
   - Clear risk communication
   - Granular control

3. **Ship and Iterate** (Pragmatist perspective)
   - Focus on core value (fresh docs)
   - Defer complex features to v2
   - Real usage data beats speculation

## Tips

### Get Better Results
- Be specific: target single files for focused updates
- Use `--dry-run` first to understand impact
- Review high-risk changes carefully

### CI Integration
```bash
# Apply only safe changes in CI
/update src/ --yes --max-risk low --no-backup
```

### After Updating
1. Run your test suite
2. Review any rejected changes manually
3. Delete backups after confirming changes work
4. Commit with a clear message about what was updated

## Troubleshooting

### "Library not found"
- Check the exact npm package name
- Try `--lib [exact-name]`
- Some libraries may not be indexed

### "No documentation available"
- The library may not be in Context7
- Try the package's canonical name
- Report missing libraries for indexing

### "Changes don't look right"
- Use `--dry-run` to preview safely
- Reject changes you disagree with
- Check documentation reference for context

## How It's Different from Regular Claude

| Regular Claude | /update Skill |
|---------------|---------------|
| Training data may be outdated | Fetches current docs at runtime |
| Suggests patterns from old versions | Knows about breaking changes |
| Can't reference migration guides | Links to official documentation |
| May hallucinate deprecated APIs | Verifies APIs exist in docs |

## Contributing

This skill emerged from a multi-agent brainstorm debate. See `DESIGN.md` for the full rationale and synthesis process.

Improvements welcome:
- Better library detection patterns
- More Context7 integrations
- Improved diff formatting
- Additional safety checks

## License

Open source - use, modify, share freely.

## Credits

Synthesized by three agents:
- **Pragmatist**: Ship value fast, defer complexity
- **Perfectionist**: Safety and verification
- **User Advocate**: Developer experience

Orchestrated via Claude Code Task tool multi-agent brainstorm pattern.
