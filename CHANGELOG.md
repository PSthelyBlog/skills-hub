# Skills Hub Changelog

All notable changes to the **skills-hub** repository will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.3.1] - 2025-01-30

### Fixed
- **skills-hub**: Corrected skill path references from `./skills-hub/[skill-name]/` to `./[skill-name]/` in SKILL.md and DESIGN.md
- **verify**: Fixed self-reference path in INSTALL.md from `./verify/SKILL.md` to `./SKILL.md`
- **README.md**: Fixed install example path from `skills-hub/brainstorm/INSTALL.md` to `brainstorm/INSTALL.md`

## [1.3.0] - 2025-01-30

### Changed
- **grounded → verify**: Renamed skill from `grounded` to `verify` for clearer action verb naming
- Updated root README.md skill table and examples
- Updated skills-hub routing to use `verify`

### Rationale
The rename emerged from a three-agent brainstorm reaching consensus that "verify" is:
- A clear action verb matching the skill naming pattern (explain, suggest, brainstorm)
- More discoverable — users searching for verification find it
- Accurately describes the core function: verifying volatile knowledge

## [1.2.0] - 2025-01-29

### Changed
- **brainstorm**: Rewrote SKILL.md using write-for-claude optimization protocol
- **verify** (formerly grounded): Rewrote SKILL.md using write-for-claude optimization protocol
- **skills-hub**: Rewrote SKILL.md using write-for-claude optimization protocol
- **suggest**: Rewrote SKILL.md using write-for-claude optimization protocol
- **update**: Rewrote SKILL.md using write-for-claude optimization protocol
- **explain**: Updated to v2.0.0 with enhanced write-for-claude protocol

All rewritten skills now include:
- MUST/SHOULD/MAY directives replacing prose suggestions
- Explicit IF/WHEN/ELSE triggers and decision trees
- Definitions, Edge Cases, Anti-Patterns, and Validation sections
- Removal of hedge words, time estimates, and philosophical sections

## [1.1.0] - 2025-01-28

### Added
- **verify** (formerly grounded): DESIGN.md, DELIVERABLES.md, QUICKSTART.txt (completing documentation set)
- **write-for-claude**: DESIGN.md, DELIVERABLES.md, QUICKSTART.txt (completing documentation set)
- CHANGELOG.md files for all skills
- Root CHANGELOG.md for repository-wide changes

### Changed
- Updated author from "Multi-Agent Synthesis" to "PSthelyBlog" across all skills
- Standardized all skills to version 1.1.0
- Updated root README.md architecture section to include all documentation files

### Fixed
- Documentation consistency: all skills now have complete file sets
- Version consistency: all skills now at 1.1.0

## [1.0.0] - 2025-01-01

### Added
- Initial release of skills-hub repository
- **brainstorm**: Multi-agent deliberation skill
- **explain**: Code explanation skill with progressive depth
- **verify** (formerly grounded): Epistemic calibration skill
- **skills-hub**: Meta-skill for routing and orchestration
- **suggest**: Code improvement suggestions skill
- **update**: Code modernization skill
- **write-for-claude**: Claude-optimized content generation skill

---

## Individual Skill Changelogs

Each skill maintains its own changelog:

- [brainstorm/CHANGELOG.md](./brainstorm/CHANGELOG.md)
- [explain/CHANGELOG.md](./explain/CHANGELOG.md)
- [verify/CHANGELOG.md](./verify/CHANGELOG.md)
- [skills-hub/CHANGELOG.md](./skills-hub/CHANGELOG.md)
- [suggest/CHANGELOG.md](./suggest/CHANGELOG.md)
- [update/CHANGELOG.md](./update/CHANGELOG.md)
- [write-for-claude/CHANGELOG.md](./write-for-claude/CHANGELOG.md)
