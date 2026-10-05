# Change Log

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](http://keepachangelog.com/)
and this project adheres to [Semantic Versioning](http://semver.org/).

## [Unreleased]

### Added

- Skills `/commit`, `/push` and `/project-setup` for Alliance Auth apps,
  taken over from a personal setup and made machine-independent: paths,
  WSL use and the database rule now come from a `## AA environment` block in
  `~/.claude/CLAUDE.md` instead of being written into the skills.
- `aa-test`, the quiet test runner the `## Release` blocks call, configurable
  through `AA_INSTANCE` and `AA_PYTHON`. Returns the exit code of the test
  run rather than of the log filter.
- Templates for `CLAUDE.md`, `CHANGELOG.md`, `docs/HANDOVER.md` and the
  `## AA environment` block.
