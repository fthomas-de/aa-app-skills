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
- `/push` tags the release commit `v<version>` ("Release <version>") and
  pushes the branch and that one tag together, so the READMEs'
  `pip install git+...@v<version>` works; an existing tag stops it.
- Every `/commit` bumps the version (default +0.0.1) and moves the running
  CHANGELOG entries onto it; `/push` no longer does either and tags the
  version at HEAD.
- `aa-test`, the quiet test runner the `## Release` blocks call, configurable
  through `AA_INSTANCE` and `AA_PYTHON`. Returns the exit code of the test
  run rather than of the log filter. Each project gets its own test database
  (`test_<repo name>` through `EOS_TEST_DB`), so parallel runs in different
  projects do not interrupt each other.
- Templates for `CLAUDE.md`, `CHANGELOG.md`, `docs/HANDOVER.md` and the
  `## AA environment` block.

### Fixed

- `/commit` and `/push` delete `.git/CLAUDE_COMMIT_MSG` after a successful
  commit and commit only once the Write succeeded. A leftover file made the
  Write fail ("File has not been read yet") and git committed the previous
  message.
