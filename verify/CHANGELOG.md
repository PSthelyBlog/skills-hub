# Changelog

All notable changes to the **verify** skill will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.3.1] - 2025-01-30

### Fixed
- Fixed self-reference path in INSTALL.md from `./verify/SKILL.md` to `./SKILL.md`

## [1.3.0] - 2025-01-30

### Changed
- **Renamed skill from `grounded` to `verify`** — action verb naming for clarity and discoverability
- Updated all documentation to use `/verify` command
- Updated skill metadata (name, examples) in SKILL.md

### Rationale
The rename emerged from a three-agent brainstorm that reached consensus:
- "Verify" is a clear action verb matching the skill naming pattern (explain, suggest, brainstorm)
- More discoverable — users searching for verification will find it
- Accurately describes the core function: verifying volatile knowledge

## [1.2.0] - 2025-01-29

### Changed
- Rewrote SKILL.md using write-for-claude protocol
- Replaced prose with MUST/SHOULD/MAY directives
- Converted "Five Questions" to decision tree with IF/ELSE branches
- Converted domain guidelines to explicit VERIFY/TRUST tables
- Added explicit volatility signal detection table

### Added
- Definitions section (VOLATILE KNOWLEDGE, STABLE KNOWLEDGE, VOLATILITY SIGNAL, HIGH-STAKES QUERY, KNOWLEDGE CUTOFF)
- Edge Cases section with WHEN conditions and DEFAULT
- Anti-Patterns section with WRONG/RIGHT examples
- Validation checklist
- Tool Selection table with explicit rationale

### Removed
- "The Problem" section (context, not instruction)
- "Core Philosophy" section (converted to decision tree)
- "The Meta-Principle" section (philosophical, not actionable)
- Lengthy examples (trimmed to Anti-Patterns comparisons)
- "Success Metrics" prose (converted to Validation checklist)

## [1.1.0] - 2025-01-28

### Added
- DESIGN.md with design rationale
- DELIVERABLES.md with deliverables summary
- QUICKSTART.txt quick reference card

### Changed
- Updated author from "Multi-Agent Synthesis" to "PSthelyBlog"
- Standardized version to 1.1.0 across all skills

### Fixed
- Missing documentation files (DESIGN.md, DELIVERABLES.md, QUICKSTART.txt)
- Version consistency with other skills in the hub

## [1.0.0] - 2025-01-01

### Added
- Initial release
- Knowledge Volatility Spectrum framework
- Five domain-specific guidelines
- Tool integration (Context7, WebSearch, WebFetch)
- Knowledge Assessment output format
- High-stakes query handling with disclaimers
- "Five Questions" decision checklist
- Verify vs. trust decision framework
