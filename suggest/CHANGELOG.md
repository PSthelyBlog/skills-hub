# Changelog

All notable changes to the **suggest** skill will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

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
