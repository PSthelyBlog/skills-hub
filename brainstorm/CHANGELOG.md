# Changelog

All notable changes to the **brainstorm** skill will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.2.0] - 2025-01-28

### Changed
- Rewrote SKILL.md using write-for-claude optimization protocol
- Replaced suggestions with MUST/SHOULD/MAY directives
- Converted prose explanations to structured directive format
- Added explicit WHEN/FOR/IF triggers to all directives

### Added
- Usage Decision Tree with IF/ELSE structure
- Anti-Patterns section with INSTEAD alternatives
- Definitions section (PARALLEL EXECUTION, SYNTHESIS, AGENT ISOLATION)
- Edge Cases section with DEFAULT fallback

### Removed
- Time estimates (per no-time-estimates principle)
- Hedge words (generally, might, try to)
- Narrative prose and flowchart diagrams
- Example interaction section

## [1.1.0] - 2025-01-28

### Changed
- Updated author from "Multi-Agent Synthesis" to "PSthelyBlog"
- Standardized version to 1.1.0 across all skills

### Fixed
- Version consistency with other skills in the hub

## [2.0.0] - 2025-01-15

### Changed
- **BREAKING**: Main session must now directly orchestrate parallel Task calls
- Corrected architecture based on empirical testing
- Task tool unavailable to subagents (prevents infinite recursion)

### Fixed
- Theatrical fallback issue where single model pretended to be multiple agents
- Skills-hub delegation no longer attempts to spawn nested Task calls

### Added
- Critical Architecture Note section in SKILL.md
- v1.0 vs v2.0 comparison table in DESIGN.md
- Troubleshooting for detecting theatrical fallback

## [1.0.0] - 2025-01-01

### Added
- Initial release
- Three-agent debate system (Pragmatist, Perfectionist, User Advocate)
- 7 preset personas
- Custom `--agents` flag
- `--transcript` flag for saving full debate
- Structured output format
- Two-round debate protocol
