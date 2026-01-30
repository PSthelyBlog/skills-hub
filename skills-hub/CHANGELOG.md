# Changelog

All notable changes to the **skills-hub** skill will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.3.0] - 2025-01-30

### Changed
- Renamed `grounded` skill to `verify` in all references
- Updated routing decision tree, skill tables, and examples to use `verify`

## [1.2.0] - 2025-01-29

### Changed
- Rewrote SKILL.md using write-for-claude protocol
- Replaced prose with MUST/SHOULD/MAY directives
- Added Routing Decision Tree with explicit IF/ELSE branches
- Added Execution Decision Tree for delegation logic
- Consolidated Available Skills table with Delegatable column

### Added
- Definitions section (SKILL, CHAIN, DELEGATION, ROUTING)
- Edge Cases section including brainstorm-in-middle-of-chain handling
- Anti-Patterns section with WRONG/RIGHT examples
- Validation checklist
- Skill Reference table with triggers, flags, and models

### Removed
- "Purpose" section (redundant with description)
- "Core Philosophy" section (converted to explicit rules)
- "Common Workflows" section (examples in frontmatter sufficient)
- "Success Metrics" prose (converted to Validation checklist)
- "Remember" closing note (philosophical, not actionable)

## [1.1.0] - 2025-01-28

### Changed
- Updated author from "Multi-Agent Synthesis" to "PSthelyBlog"
- Standardized version to 1.1.0 across all skills

### Fixed
- Version consistency with other skills in the hub

## [1.0.0] - 2025-01-01

### Added
- Initial release
- Intelligent skill routing based on intent
- Skill chaining with "then" keyword
- Parallel execution for independent skills
- Context isolation via Task subagents
- Support for all skills: explain, suggest, brainstorm, update, verify, write-for-claude
- Special handling for brainstorm (cannot be delegated)
- Result synthesis for chained/parallel workflows
