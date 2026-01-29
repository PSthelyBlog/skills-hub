# Changelog

All notable changes to the **update** skill will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.2.0] - 2025-01-29

### Changed
- Rewrote SKILL.md using write-for-claude protocol
- Replaced prose with MUST/SHOULD/MAY directives
- Added Detection Decision Tree for library identification
- Added explicit import pattern detection table
- Formalized risk level criteria with examples
- Replaced emoji risk icons with text markers ([LOW]/[MED]/[HIGH])

### Added
- Definitions section (MIGRATION, BREAKING CHANGE, RISK LEVEL, BACKUP, DRY RUN)
- Edge Cases section (library not found, no changes needed, partial failure, Context7 unavailable, scope all)
- Anti-Patterns section with WRONG/RIGHT examples
- Validation checklist
- Explicit Context7 tool call syntax

### Removed
- "Purpose" section (redundant with description)
- "Core Problem Solved" ASCII diagram (context, not instruction)
- "How It Works" overview (redundant with Protocol)
- "Model Selection" explanation (model specified in frontmatter)
- "Integration with Context7" section (integrated into protocol steps)
- "Remember" closing note (philosophical)

## [1.1.0] - 2025-01-28

### Changed
- Updated author from "Multi-Agent Synthesis" to "PSthelyBlog"
- Standardized version to 1.1.0 across all skills

### Fixed
- Version consistency with other skills in the hub

## [1.0.0] - 2025-01-01

### Added
- Initial release
- Fresh documentation fetching via Context7
- Library auto-detection from imports
- Risk classification (low, medium, high)
- Per-change approval workflow
- Timestamped backup system
- Audit logging in `.update-audit/`
- `--lib` flag for specific library
- `--scope` flag (function/file/all)
- `--dry-run` flag for preview
- `--verbose` flag for detailed output
- `--yes` flag for skip confirmation
- `--max-risk` flag for risk filtering
- `--no-backup` flag option
