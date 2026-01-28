# Design Rationale: The Update Skill

## Synthesis Overview

This skill emerged from a three-agent brainstorm that balanced competing values:

| Agent | Core Value | Key Concern |
|-------|------------|-------------|
| **Pragmatist** | Ship fast, defer complexity | Don't over-engineer the first version |
| **Perfectionist** | Safety and verification | Never apply unverified changes |
| **User Advocate** | Developer experience | Don't require expert knowledge |

## The Core Problem

AI assistants have training data cutoffs. When helping modernize code:
- They suggest patterns from old library versions
- They miss breaking changes in recent releases
- They can't reference current documentation

**The insight**: We can inject fresh documentation at decision time. The AI's reasoning is still valuable; it just needs current facts to reason about.

## How Each Perspective Shaped the Design

### From the Pragmatist

**Core argument**: "Ship value fast. Real usage data beats speculation."

**Contributions**:
- Focus MVP on core value proposition (fresh docs)
- Defer complex features (changelog parsing, confidence scores) to v2
- Use existing infrastructure (Context7) rather than building from scratch
- Default to sensible behaviors rather than extensive configuration

**Visible in**:
- Single command to get started (`/update [file]`)
- Auto-detection of libraries (no manual specification needed)
- Reasonable defaults for all options
- Lightweight audit logging (JSON lines, not database)

**What they pushed back on**:
- Full changelog parsing: "We don't know what formats we'll encounter"
- Confidence scoring: "Need usage data first to calibrate"
- Multi-step wizards: "Just show a diff and ask yes/no"

### From the Perfectionist

**Core argument**: "Code changes can break production. Safety is non-negotiable."

**Contributions**:
- Never apply without showing diff first
- Always create backup before modifying
- Verify proposed APIs exist in fetched docs
- Comprehensive audit logging for debugging
- Per-change approval workflow

**Visible in**:
- `--dry-run` flag for safe preview
- Automatic backup to `.update-backups/`
- Risk level classification (low/medium/high)
- Breaking change warnings (⚠️ flags)
- Audit logs in `.update-audit/`

**What they pushed back on**:
- `--yes` flag initially: "Too dangerous for code changes"
  - *Compromise*: Allow `--yes` but it's opt-in, not default
- Trusting all AI suggestions: "Must cross-reference with docs"
- Skipping confirmation: "User must see what will change"

### From the User Advocate

**Core argument**: "Junior devs shouldn't need to know library IDs. Meet users where they are."

**Contributions**:
- Auto-detect libraries from imports
- Clear risk communication (color-coded levels)
- Granular control (accept/reject per change)
- Helpful error messages with suggestions
- Progressive workflow (preview → approve → apply)

**Visible in**:
- Library detection from `import` statements
- `--lib` as override, not requirement
- "Did you mean?" suggestions for unknown libraries
- Summary view before detailed changes
- Accept All / Reject All options

**What they pushed back on**:
- Requiring `--lib` flag: "Friction kills adoption"
- Technical error messages: "Say what to do, not what failed"
- All-or-nothing changes: "Let me pick which changes I want"

## Key Design Decisions

### 1. Auto-Detection with Confirmation

**Debate**:
| Position | Agent | Argument |
|----------|-------|----------|
| Require `--lib` | Pragmatist (initial), Perfectionist | Explicit is predictable |
| Auto-detect only | User Advocate | Zero-friction UX |
| **Hybrid** | All (final) | Auto-detect, confirm, allow override |

**Resolution**: Scan imports, prompt user to confirm, allow `--lib` to skip detection.

**Why this works**:
- Low friction: Just run `/update [file]`
- Safe: User confirms before fetching docs
- Escape hatch: Power users can specify exactly

### 2. Risk Level Classification

**Purpose**: Help users understand impact without reading every diff.

**Levels**:
- 🟢 **Low**: Safe renames, import path changes
- 🟡 **Medium**: API signature changes (same behavior)
- 🔴 **High**: Behavioral changes (caching, timing, etc.)

**Breaking flag** (⚠️): Applied regardless of risk level when the change affects compatibility.

**Decision**: Classification happens at proposal time based on:
- Keywords in documentation ("breaking", "deprecated", "removed")
- Type of change (import vs. function body)
- Scope of change (single line vs. multi-line)

### 3. Per-Change Approval

**Perfectionist concern**: "All-or-nothing is too coarse"
**Pragmatist concern**: "Too many prompts kills UX"

**Resolution**: Three approval modes:
1. **Accept All**: For confident users
2. **Review Each**: Default for safety
3. **Accept All Low-Risk**: Best of both worlds

The `--max-risk` flag pre-filters, so users only review what matters to them.

### 4. Backup and Audit

**All agents agreed**: Code changes need safety nets.

**Backup strategy**:
```
.update-backups/[filename].[timestamp].bak
```
- Created before any modification
- Path included in completion message
- User must explicitly skip with `--no-backup`

**Audit strategy**:
```
.update-audit/YYYY-MM-DD.jsonl
```
- Append-only JSON lines (simple, portable)
- Captures: detection, fetch, proposal, apply, reject, error
- Session ID links related operations

### 5. Context7 as Truth Source

**Pragmatist**: "Use existing infrastructure, don't rebuild docs"
**Perfectionist**: "We need authoritative sources"
**User Advocate**: "Users shouldn't manage doc sources"

**Resolution**: Context7 is the single source of truth for documentation.

**Trade-offs**:
- Pro: Fresh docs without building indexing infrastructure
- Con: Dependent on external service
- Con: Not all libraries indexed

**Mitigation**: Clear error when library not found, with suggestions.

## What We Didn't Build (v1)

### Deferred: Changelog Parsing

**Idea**: Parse CHANGELOG.md, GitHub releases, etc. for breaking changes.

**Why deferred**:
- Formats are inconsistent
- Would need to support multiple formats
- Context7 often includes breaking change info
- Can add in v2 based on real gaps

### Deferred: Confidence Scoring

**Idea**: Rate each suggestion (high/medium/low confidence).

**Why deferred**:
- Need usage data to calibrate
- Risk levels serve similar purpose
- Adds complexity without proven value

### Deferred: Multi-File Coordination

**Idea**: Batch changes across files with consolidated preview.

**Why deferred**:
- Complex state management
- Partial failure handling unclear
- `--scope all` covers basic case

### Rejected: Automatic Model Selection

**Idea**: Use Haiku for simple updates, Sonnet for complex.

**Why rejected**:
- Code modification is always high-stakes
- Consistency matters more than cost savings
- Sonnet for all updates is acceptable

## Comparison to /explain Skill

The `/explain` skill used progressive disclosure (quick vs deep modes).

The `/update` skill uses progressive approval instead:
- **Preview** → See what would change
- **Confirm** → Approve libraries and changes
- **Apply** → Execute with backup

Both skills share:
- Three-agent synthesis origin
- Safety-first principles
- Clear user communication
- Structured output formats

Key difference:
- `/explain` is read-only (safe to be fast)
- `/update` modifies files (safety requires friction)

## Success Criteria

This skill succeeds if:

1. **Users trust it** — They run `/update` without fear
2. **Changes are accurate** — Based on real docs, not hallucinations
3. **Recovery is easy** — Backups and audit make mistakes reversible
4. **Adoption grows** — Friction is low enough for regular use
5. **False positives are rare** — Proposed changes are usually correct

## Lessons for Future Skills

### 1. Safety and UX aren't opposites
The Perfectionist and User Advocate found common ground: users want safety, just not at the cost of usability. Per-change approval with "Accept All" satisfies both.

### 2. External data solves training cutoffs
The core insight applies beyond library updates. Any skill that needs current information can inject it at runtime.

### 3. Hybrid beats extremes
Auto-detect vs require flag. Accept all vs review each. The hybrid approach (detect then confirm) serves both experts and novices.

### 4. Defer wisely
The Pragmatist's "ship then iterate" prevented scope creep. Real usage data will inform v2 features better than speculation.

### 5. Documentation-only skills work
This skill has no code—just instructions. Claude's reasoning capabilities, combined with existing tools (Context7, Read, Edit), execute the workflow.

## Open Questions for v2

### Scope Expansion
- Should `--scope project` scan all files automatically?
- How to handle monorepos with multiple package.json files?

### Breaking Change Detection
- Should we parse changelogs for additional context?
- How to surface "soft" deprecations (not removed, but discouraged)?

### CI Integration
- What's the right default for unattended runs?
- Should we support GitHub Actions annotations?

### Multi-Language Support
- How to detect libraries in Python (requirements.txt, pyproject.toml)?
- Different ecosystems have different update patterns

## Conclusion

The `/update` skill demonstrates that safety and usability can coexist. By:

- **Auto-detecting** (User Advocate) then **confirming** (Perfectionist)
- **Showing diffs** (Perfectionist) with **clear risk levels** (User Advocate)
- **Using existing infra** (Pragmatist) with **comprehensive logging** (Perfectionist)

...we created a skill that developers can trust with their code.

The three perspectives didn't compromise—they composed. Each made the skill better:

- Pragmatist kept it shippable
- Perfectionist made it safe
- User Advocate made it usable

**That's synthesis in action.**

---

*This design document captures decisions as of v1.0.0. Future versions may revisit these choices based on real-world usage.*
