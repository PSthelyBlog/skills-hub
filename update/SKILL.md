# Update Skill

---
name: update
description: "Modernize code using fresh documentation to overcome AI training cutoffs"
version: 1.1.0
author: PSthelyBlog
tags: [modernization, libraries, documentation, migration]
model: sonnet
examples:
  - "/update src/components/UserProfile.tsx"
  - "/update src/api/users.ts --lib express"
  - "/update src/hooks/useAuth.ts --dry-run"
  - "/update src/utils.ts --scope all"
---

## Purpose

The `/update` skill injects fresh documentation at decision time, enabling accurate code modernization despite AI training cutoffs. Instead of suggesting outdated patterns, it fetches current library documentation and proposes changes based on the latest APIs.

## Core Problem Solved

```
AI Training Cutoff ────────────────► Today
       │                                │
       │    Libraries released here     │
       │    are unknown to the AI       │
       ▼                                ▼
   React 17 docs                    React 19 exists
   Express 4 patterns               Express 5 released
```

**Solution**: Fetch documentation at runtime via Context7, ensuring suggestions reflect current APIs.

## How It Works

1. **Detects libraries** from imports in the target file
2. **Confirms** with user which libraries to update
3. **Fetches fresh docs** from Context7
4. **Generates migration diff** comparing current code to modern patterns
5. **Previews changes** with risk levels and breaking change flags
6. **Applies approved changes** with backup

## Instructions for Claude

When the user invokes `/update [target] [options]`, follow this protocol:

### 1. Parse the Request

Extract from the command:
- **Target**: File path to update (required)
- **--lib**: Specific library to update (optional, auto-detect if omitted)
- **--scope**: `function`, `file` (default), or `all`
- **--dry-run**: Preview without applying
- **--verbose**: Show detailed output
- **--yes**: Skip confirmation prompts
- **--max-risk**: `low`, `medium`, or `high` (default)
- **--no-backup**: Skip backup creation

### 2. Read the Target File

First, read the file that needs updating:

```
Use the Read tool to get the contents of [target file]
```

If the file doesn't exist or can't be read, provide a clear error:
```
Could not read [target]. Please verify the path is correct.
```

### 3. Detect Libraries

**If --lib is specified:**
Use the specified library directly.

**If --lib is NOT specified:**
Scan the file for import statements:

```javascript
// Detect these patterns:
import React from 'react';              // → react
import { useState } from 'react';       // → react
import * as lodash from 'lodash';       // → lodash
import express from 'express';          // → express
const zod = require('zod');             // → zod
```

Look for:
- ES6 imports (`import ... from '...'`)
- CommonJS requires (`require('...')`)
- Dynamic imports (`import('...')`)

Also check `package.json` in the project root for version information.

**Present detected libraries to user:**

```
## Detected Libraries

📦 react@17.0.2
📦 @tanstack/react-query@3.39.0
📦 zod@3.21.0

Update these libraries? [Y/n/edit]
```

If `--yes` flag is present, skip this confirmation.

### 4. Fetch Fresh Documentation

For each confirmed library, use Context7 to fetch current documentation:

**Step 4a: Resolve library ID**
```
Use mcp__plugin_context7_context7__resolve-library-id with:
- libraryName: [detected library name]
- query: "migration guide API changes [library name]"
```

**Step 4b: Query documentation**
```
Use mcp__plugin_context7_context7__query-docs with:
- libraryId: [resolved ID from step 4a]
- query: "migration from [current version] breaking changes new API"
```

**If documentation fetch fails:**
```
⚠️ Could not fetch documentation for [library].
Reason: [error message]

Options:
1. Try specifying with --lib [exact-name]
2. Skip this library and continue
3. Cancel the update
```

### 5. Generate Proposed Changes

Compare the current code patterns against the fetched documentation to identify:

**Types of changes:**

| Risk Level | Icon | Description | Example |
|------------|------|-------------|---------|
| Low | 🟢 | Safe renames, import changes | `react-query` → `@tanstack/react-query` |
| Medium | 🟡 | API signature changes | `useQuery(key, fn)` → `useQuery({ queryKey, queryFn })` |
| High | 🔴 | Behavioral changes | Caching strategy differences |

**Breaking changes** are flagged with ⚠️ regardless of risk level.

**Generate a unified diff for each change:**

```diff
// Change 1 of 3 (🟢 Low risk)
// Reason: Package renamed in v5

- import { useQuery } from 'react-query'
+ import { useQuery } from '@tanstack/react-query'

// Documentation reference: https://tanstack.com/query/latest/docs/react/guides/migrating-to-v5
```

### 6. Preview Changes

**Default format** (summary view):

```
## Update Opportunities for [filename]

### @tanstack/react-query v5 Migration
⚠️ BREAKING: Package renamed, API changes

📊 Summary:
- 🟢 2 low-risk changes (import updates)
- 🟡 3 medium-risk changes (API signatures)
- 🔴 1 high-risk change (caching behavior)

[Show Details] [Accept All Low-Risk] [Review Each]
```

**Verbose format** (--verbose or --dry-run):

Show each change individually with:
- Line numbers affected
- Before/after code
- Risk level and reasoning
- Documentation reference

### 7. Get User Approval

**Unless --yes is specified**, prompt for each change:

```
Change 1 of 6: Import path update (🟢 Low)

- import { useQuery } from 'react-query'
+ import { useQuery } from '@tanstack/react-query'

[Accept] [Reject] [Accept All] [Reject All] [Show Context]
```

Track approved and rejected changes.

### 8. Apply Changes

**Before applying:**
1. Create a backup (unless --no-backup):
   ```
   Backup created: .update-backups/[filename].[timestamp].bak
   ```

2. Apply changes in reverse line order (to preserve line numbers)

3. Verify the file was written correctly

**After applying:**

```
## Update Complete

✅ Applied 5 changes to [filename]
❌ Rejected 1 change (user choice)

📁 Backup: .update-backups/src_components_UserProfile.tsx.1706140800.bak

### Applied Changes:
1. ✅ Import path: react-query → @tanstack/react-query
2. ✅ useQuery signature updated
3. ✅ useMutation signature updated
4. ✅ QueryClient configuration
5. ✅ Cache time option renamed

### Rejected:
6. ❌ Caching strategy change (user rejected)

Next steps:
- Run your tests to verify the changes
- Review the rejected change manually if needed
- Delete backup after confirming changes work
```

### 9. Audit Logging

Log every operation to `.update-audit/YYYY-MM-DD.jsonl`:

```json
{
  "timestamp": "2026-01-24T12:00:00Z",
  "operation": "apply",
  "targetFile": "src/hooks/useAuth.ts",
  "libraries": ["@tanstack/react-query"],
  "details": {
    "proposed": 6,
    "applied": 5,
    "rejected": 1,
    "backupPath": ".update-backups/src_hooks_useAuth.ts.1706140800.bak"
  },
  "sessionId": "update-1706140800-abc123"
}
```

### 10. Error Handling

**Library not found:**
```
Library "[name]" not found in Context7.

Suggestions:
- Check if this is the npm package name
- Try alternative names: [suggest similar libraries]
- Use --lib with the exact package name
```

**No changes needed:**
```
## No Updates Found

Your code already follows current [library] patterns.
Version: [current] → [latest]

No changes needed!
```

**Partial failure:**
```
## Partial Update

✅ 3 changes applied successfully
⚠️ 2 changes failed to apply

Failed changes have been logged. Your code may be in an inconsistent state.
Backup available at: [backup path]

Would you like to restore from backup? [Y/n]
```

## Scope Options

| Scope | Description | Use When |
|-------|-------------|----------|
| `function` | Update only the function at cursor | Making targeted changes |
| `file` | Update the entire file (default) | Modernizing a single file |
| `all` | Update all files importing the library | Project-wide migration |

## Risk Level Filtering

Use `--max-risk` to limit which changes are proposed:

```bash
# Only safe renames and import changes
/update src/api/ --max-risk low

# Include API signature changes
/update src/api/ --max-risk medium

# Include behavioral changes (default)
/update src/api/ --max-risk high
```

## Safety Principles

**Always:**
- Show diff before applying
- Create backup by default
- Log all operations
- Allow per-change approval
- Verify proposed APIs exist in fetched docs

**Never:**
- Apply changes without preview
- Hallucinate APIs not in documentation
- Skip backup without explicit --no-backup
- Ignore breaking change warnings

## Model Selection

This skill uses Sonnet by default because:
- Code analysis requires understanding patterns
- Diff generation needs precision
- Breaking change detection is complex
- Safety is paramount for code modifications

Do not downgrade to Haiku for this skill.

## Integration with Context7

The skill relies on Context7 for fresh documentation. If Context7 is unavailable:

```
⚠️ Documentation service unavailable

Cannot proceed without fresh documentation (would rely on potentially outdated training data).

Options:
1. Retry fetching documentation
2. Cancel update
3. Proceed without docs (not recommended, may suggest outdated patterns)
```

---

**Remember**: The goal is accurate modernization, not just any changes. Only propose changes backed by fetched documentation. When in doubt, show the user and let them decide.
