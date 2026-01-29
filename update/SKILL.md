# Update Skill

---
name: update
description: "Modernize code using fresh Context7 documentation to overcome AI training cutoffs"
version: 1.2.0
author: PSthelyBlog
tags: [modernization, libraries, documentation, migration]
model: sonnet
examples:
  - "/update src/components/UserProfile.tsx"
  - "/update src/api/users.ts --lib express"
  - "/update src/hooks/useAuth.ts --dry-run"
  - "/update src/utils.ts --scope all"
---

## Trigger

WHEN user invokes `/update [target] [options]`: Execute this protocol.

## Protocol

### Step 1: Parse Request

Extract from user input:

| Component | Flag | Default | Valid values |
|-----------|------|---------|--------------|
| Target | positional | none (required) | File path |
| Library | `--lib` | auto-detect | Package name |
| Scope | `--scope` | file | function, file, all |
| Dry run | `--dry-run` | off | on/off |
| Verbose | `--verbose` | off | on/off |
| Skip prompts | `--yes` | off | on/off |
| Max risk | `--max-risk` | high | low, medium, high |
| Skip backup | `--no-backup` | off | on/off |

IF target not provided:
  Ask: "What file should I update?"

### Step 2: Read Target File

MUST: Use Read tool to get contents of target file

IF file does not exist:
  Return error: "Could not read [target]. Please verify the path is correct."

IF file is empty:
  Return error: "File [target] is empty. Nothing to update."

### Step 3: Detect Libraries

#### 3.1 Detection Decision Tree

```
IF --lib flag specified:
  Use specified library directly
ELSE:
  Scan file for import patterns
  Check package.json for versions
  Present detected libraries for confirmation
```

#### 3.2 Import Pattern Detection

Scan for these patterns:

| Pattern | Example | Extracts |
|---------|---------|----------|
| ES6 default | `import React from 'react'` | react |
| ES6 named | `import { useState } from 'react'` | react |
| ES6 namespace | `import * as lodash from 'lodash'` | lodash |
| CommonJS | `const zod = require('zod')` | zod |
| Dynamic | `import('express')` | express |

MUST: Check `package.json` in project root for version information
MUST: Exclude Node.js built-ins (fs, path, http, etc.)
MUST: Exclude relative imports (./utils, ../components)

#### 3.3 User Confirmation

IF `--yes` flag NOT present:

```markdown
## Detected Libraries

- [library]@[version]
- [library]@[version]

Update these libraries?
```

Use AskUserQuestion with options:
- "Yes, update all"
- "Select specific libraries"
- "Cancel"

IF "Select specific libraries": Present checkboxes for each library

### Step 4: Fetch Documentation

FOR each confirmed library:

#### 4.1 Resolve Library ID

```
mcp__plugin_context7_context7__resolve-library-id(
  libraryName: [library name],
  query: "migration guide API changes [library name]"
)
```

IF resolution fails:
  Log warning, continue with next library

#### 4.2 Query Documentation

```
mcp__plugin_context7_context7__query-docs(
  libraryId: [resolved ID],
  query: "migration from [current version] breaking changes new API"
)
```

IF documentation fetch fails:

```markdown
Could not fetch documentation for [library].
Reason: [error]

Options:
1. Try with --lib [exact-name]
2. Skip this library
3. Cancel update
```

Use AskUserQuestion for user choice.

### Step 5: Generate Proposed Changes

#### 5.1 Change Analysis

Compare current code against fetched documentation to identify:

| Risk Level | Icon | Criteria | Examples |
|------------|------|----------|----------|
| Low | `[LOW]` | No behavior change, safe renames | Import path changes, type renames |
| Medium | `[MED]` | API signature changes, same behavior | Function parameter restructuring |
| High | `[HIGH]` | Behavioral changes, new semantics | Caching strategy, default values |

MUST: Flag breaking changes with `[BREAKING]` regardless of risk level
MUST: Only propose changes backed by fetched documentation
MUST NOT: Propose changes based on training data alone

#### 5.2 Risk Filtering

IF `--max-risk low`: Include only LOW risk changes
IF `--max-risk medium`: Include LOW and MED risk changes
IF `--max-risk high` (default): Include all changes

#### 5.3 Change Format

FOR each change, generate:

```markdown
### Change N of M: [Brief title] [RISK] [BREAKING if applicable]

**Reason:** [One sentence from documentation]

**Before:**
```language
[exact code from file]
```

**After:**
```language
[updated code]
```

**Documentation:** [URL or reference]
```

### Step 6: Preview Changes

#### 6.1 Summary View (Default)

```markdown
## Update Opportunities for [filename]

### [Library] Migration

[BREAKING] if any breaking changes

**Summary:**
- [LOW] N changes (import updates, renames)
- [MED] N changes (API signatures)
- [HIGH] N changes (behavioral)

Total: N changes affecting M lines
```

#### 6.2 Verbose View (`--verbose` or `--dry-run`)

Show each change with:
- Line numbers affected
- Before/after code
- Risk level and reasoning
- Documentation reference

IF `--dry-run`:
  Show all changes, then exit without applying
  Return: "Dry run complete. No changes applied."

### Step 7: Get User Approval

IF `--yes` flag present:
  Skip approval, apply all changes within risk threshold

ELSE:
  FOR each change:

```markdown
Change N of M: [Title] [RISK]

**Before:**
[code]

**After:**
[code]
```

Use AskUserQuestion with options:
- "Apply this change"
- "Skip this change"
- "Apply all remaining"
- "Skip all remaining"

Track: approved_changes[], rejected_changes[]

### Step 8: Apply Changes

#### 8.1 Pre-Apply

IF `--no-backup` NOT present:
  MUST: Create backup at `.update-backups/[filename].[timestamp].bak`
  MUST: Verify backup was created before proceeding

#### 8.2 Apply

MUST: Apply changes in reverse line order (preserves line numbers)
MUST: Use Edit tool for each change
MUST: Verify each edit succeeded before continuing

IF any edit fails:
  Stop immediately
  Report which changes succeeded and which failed
  Offer to restore from backup

#### 8.3 Post-Apply Summary

```markdown
## Update Complete

**Applied:** N changes to [filename]
**Rejected:** M changes (user choice)

**Backup:** .update-backups/[path].[timestamp].bak

### Applied Changes:
1. [Title] - [one-line description]
2. [Title] - [one-line description]

### Rejected:
1. [Title] - [reason if provided]

### Next Steps:
- Run tests to verify changes
- Review rejected changes manually if needed
- Delete backup after confirming changes work
```

### Step 9: Audit Logging

MUST: Log every operation to `.update-audit/YYYY-MM-DD.jsonl`

```json
{
  "timestamp": "[ISO timestamp]",
  "operation": "apply",
  "targetFile": "[path]",
  "libraries": ["[lib1]", "[lib2]"],
  "proposed": N,
  "applied": N,
  "rejected": N,
  "backupPath": "[path]"
}
```

## Definitions

MIGRATION: Updating code from one library version to another
BREAKING CHANGE: Change that requires modifications to calling code
RISK LEVEL: Assessment of how likely a change is to cause issues (LOW/MED/HIGH)
BACKUP: Copy of original file stored before modifications
DRY RUN: Preview mode that shows changes without applying them

## Edge Cases

WHEN library not found in Context7:

```markdown
Library "[name]" not found in Context7.

Suggestions:
- Verify this is the npm package name
- Try: [similar library names from search]
- Use --lib with exact package name
```

WHEN no changes needed:

```markdown
## No Updates Found

[library]@[current] is already using current patterns.

No changes needed.
```

WHEN code already uses latest patterns:
  Return "No updates found" message
  MUST NOT: Propose unnecessary changes

WHEN partial failure during apply:

```markdown
## Partial Update

**Applied:** N changes
**Failed:** M changes

Your code may be in an inconsistent state.
Backup available at: [path]

Restore from backup?
```

Use AskUserQuestion with options: "Restore backup", "Keep partial changes"

WHEN Context7 unavailable:

```markdown
Documentation service unavailable.

Cannot proceed without fresh documentation (would rely on potentially outdated training data).

Options:
1. Retry
2. Cancel
```

MUST NOT: Proceed without documentation (would use stale training data)

WHEN `--scope all` specified:
  MUST: Find all files importing the library
  MUST: Process each file with same protocol
  MUST: Show aggregate summary at end

DEFAULT: If uncertain about a change's risk level, classify as HIGH.

## Anti-Patterns

MUST NOT: Propose changes not backed by fetched documentation

```markdown
// WRONG: Change based on training data
"I know React 19 has a new API..."
[Proposes change without Context7 verification]

// RIGHT: Change backed by documentation
"According to Context7 docs for React 19..."
[Shows documentation reference]
```

MUST NOT: Apply changes without preview

```markdown
// WRONG: Immediate application
User: /update src/api.ts
Claude: [Applies changes directly]

// RIGHT: Preview first
User: /update src/api.ts
Claude: [Shows proposed changes, waits for approval]
```

MUST NOT: Skip backup without explicit flag

```markdown
// WRONG: No backup created
[Applies changes without backup]

// RIGHT: Backup by default
Backup created: .update-backups/[file].[timestamp].bak
[Then applies changes]
```

MUST NOT: Propose changes to code that doesn't use the library

```markdown
// WRONG: Suggesting unrelated changes
"While updating react-query, I also noticed..."
[Proposes unrelated improvements]

// RIGHT: Stay focused on library migration
[Only proposes changes related to specified/detected libraries]
```

## Flags Reference

| Flag | Description | Default |
|------|-------------|---------|
| `--lib` | Specific library to update | auto-detect |
| `--scope` | function, file, or all | file |
| `--dry-run` | Preview without applying | off |
| `--verbose` | Show detailed output | off |
| `--yes` | Skip confirmation prompts | off |
| `--max-risk` | low, medium, or high | high |
| `--no-backup` | Skip backup creation | off |

## Validation

BEFORE returning, verify:

- [ ] Target file was read successfully
- [ ] Libraries detected or specified via --lib
- [ ] Context7 documentation fetched for each library
- [ ] All proposed changes reference fetched documentation
- [ ] Risk levels assigned to all changes
- [ ] Breaking changes flagged with [BREAKING]
- [ ] Changes filtered by --max-risk threshold
- [ ] Backup created (unless --no-backup)
- [ ] Audit log entry written
- [ ] Summary shows applied/rejected counts

IF validation fails: Fix before returning.
