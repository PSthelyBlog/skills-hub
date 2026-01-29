# Changelog

All notable changes to the **suggest** skill will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.2.0] - 2025-01-29

### Changed
- Rewrote SKILL.md using write-for-claude protocol
- Replaced prose with MUST/SHOULD/MAY directives
- Added Context Decision Tree for explain chaining
- Added explicit severity assignment criteria table
- Formalized ranking rules with MUST directives

### Added
- Definitions section (SUGGESTION, SEVERITY, CATEGORY, DEPTH, APPLY MODE)
- Edge Cases section (test breaking, style conflicts, overlapping suggestions, directory targets)
- Anti-Patterns section with WRONG/RIGHT examples for common mistakes
- Validation checklist
- Explicit format rules for output sections

### Removed
- "Purpose" section (redundant with description)
- "Core Philosophy" section (converted to format rules)
- "How It Works" overview (redundant with Protocol)
- Lengthy examples (key patterns moved to Anti-Patterns)
- "Usage Patterns" section (examples in frontmatter sufficient)
- "Notes for Skill Authors" section (meta-commentary)
- "Remember" closing note

## [1.1.0] - 2025-01-28

### Changed
- Updated author from "Multi-Agent Synthesis" to "PSthelyBlog"
- Standardized version to 1.1.0 across all skills

### Fixed
- Version consistency with other skills in the hub

## [1.0.0] - 2025-01-01

### Added
- Initial release
- Four suggestion categories (bugs, security, performance, readability)
- Three depth levels (quick, standard, thorough)
- Before/after diff format for all suggestions
- Severity ranking (HIGH, MED, LOW)
- `--category` flag for filtering
- `--depth` flag for analysis depth
- `--limit` flag for suggestion count
- `--apply` flag for interactive application
- Context chaining with prior /explain
