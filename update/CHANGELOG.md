# Changelog

All notable changes to the **update** skill will be documented in this file.

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
