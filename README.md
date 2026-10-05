# aa-app-skills

Claude Code skills for developing [Alliance Auth](https://gitlab.com/allianceauth/allianceauth)
apps: a checked commit, a release with version bump and handover, and a
one-time project setup that makes both work.

| Skill | What it does |
|---|---|
| `/commit` | Test suite without translation tests, checks, translations if texts changed, translation tests, then a local commit. No version bump, no push. |
| `/push` | Version +0.0.1 (or the one you name), split `CHANGELOG.md` onto it, update `docs/HANDOVER.md` for the next session, release commit, push. Pushes without asking only when nothing stood out. |
| `/project-setup` | Sets up the machine once (`## AA environment`, `aa-test`) and each app (`CLAUDE.md` with `## Release`, `CHANGELOG.md`, `docs/HANDOVER.md`, tagged translation tests, installed into the dev instance). |

All three run only when you type them (`disable-model-invocation: true`).

## How it fits together

The skills hold no paths and no app names. They read two blocks:

- **`## AA environment`** - the machine: repos, Alliance Auth instance,
  Python, local settings, database, and whether commands run natively or
  through WSL. Lives in `~/.claude/CLAUDE.md`, so every session sees it;
  a project's `CLAUDE.local.md` can override it.
  Template: [`skills/project-setup/templates/aa-environment.md`](skills/project-setup/templates/aa-environment.md).
- **`## Release`** - the app: version file, changelog section, test,
  check and translation commands. Lives in the app's `CLAUDE.md` and is
  committed with it; a bare `python` in it means the Python from
  `## AA environment`, so it holds no machine paths.
  Template: [`skills/project-setup/templates/CLAUDE.md`](skills/project-setup/templates/CLAUDE.md).

When a command changes, change the block, not the skills.

## Requirements

- Claude Code
- a local Alliance Auth dev instance with its own venv
- git; on Windows with the instance in WSL, git runs inside WSL
- translation tests tagged `@tag("translations")` (`/project-setup` does this)

## Install

Copy (or symlink) the three skill folders into your personal skills:

```bash
git clone https://github.com/fthomas-de/aa-app-skills.git
cp -r aa-app-skills/skills/* ~/.claude/skills/
```

Then open a Claude Code session in an app repo and run `/project-setup`. The
first time it asks about your machine, writes `## AA environment` to
`~/.claude/CLAUDE.md` and installs the test runner `aa-test` to `~/bin`.

## aa-test

[`skills/project-setup/scripts/aa-test`](skills/project-setup/scripts/aa-test)
wraps `manage.py test`: drops the DEBUG/INFO lines Alliance Auth's console
logging adds (and nothing else), keeps the test database between runs
(`--keepdb`, `--fresh` rebuilds it) and returns the test run's exit code.
It finds the instance through `AA_INSTANCE` and `AA_PYTHON` (defaults
`~/aa-dev/working/myauth` and `~/aa-dev/venv/bin/python`); export them in
`~/.profile` if yours differ.

## License

See [LICENSE](LICENSE).
