---
name: project-setup
description: >
  Sets up a new or existing Alliance Auth / Django app so that /commit and
  /push work right away: the machine's "## AA environment" block, the aa-test
  runner, a CLAUDE.md with the uniform "## Release" block, a Keep-a-Changelog
  CHANGELOG, docs/HANDOVER.md, translation tests tagged
  @tag("translations"), and the app installed into the dev instance. Only on
  an explicit /project-setup.
disable-model-invocation: true
---

# Project setup (Alliance Auth / Django apps)

Goal: afterwards `/commit` and `/push` run in this project without any
adjustment. The templates next to this file (`templates/`, `scripts/`) are
the reference; when in doubt, follow them instead of inventing.

Talk to the user in their language; everything written into the project is
English.

**Never overwrite.** Existing files get extended, not replaced. If anything
is unclear (app name, languages, paths), ask with answer options instead of
guessing.

## 0 · Machine: `## AA environment`

Look for the block in context (`~/.claude/CLAUDE.md`, or the project's
`CLAUDE.local.md`). If it exists, use it and go on to step 1.

If it is missing, set it up once for this machine:

1. Find out what you can: is Claude Code running on Windows with the
   instance in WSL (`wsl.exe -l -q`), or natively? Where is `manage.py` of the
   Alliance Auth instance, where its venv's Python, which local settings file,
   which database (`DATABASES` in the settings)?
2. Show the findings and ask with options to confirm or correct them, and
   whether the dev database is **irreplaceable** (holds data that cannot be
   fetched again, such as ESI data) or disposable.
3. Fill in `templates/aa-environment.md` (WSL variant from its comment if
   needed, drop the comment) and append it to `~/.claude/CLAUDE.md` - create
   the file if it does not exist. That file is loaded into every session, so
   the block applies to all apps on this machine.

From here on run every command - git included - the way `Shell` says, and
reach files the way `Files` says. Under WSL: never set `safe.directory`.

## 0a · Test runner `aa-test`

`command -v aa-test` (through `Shell`). If it is missing, ask once, then
copy `scripts/aa-test` from this skill's directory to `~/bin/aa-test` on the
machine that runs the instance (under WSL from `/mnt/c/...`), `chmod +x` it
and check that `~/bin` is on `PATH` in a login shell. If the instance or
Python differ from the script's defaults, export `AA_INSTANCE` and
`AA_PYTHON` in `~/.profile` - not `~/.bashrc`, which returns early for the
non-interactive shells the skills use.

## 1 · Take stock

Read once, show the user briefly:

- repo path, git remote, branch
- app module (folder with `apps.py`) and version file (`<app>/__init__.py`
  with `__version__`; if missing, propose `__version__ = "0.0.1"`)
- installed in the venv (`<python> -m pip show <distribution>`) and listed in
  the local settings? Step 1a takes care of both.
- `CHANGELOG.md`, `docs/HANDOVER.md`, `CLAUDE.md` present?
- tests: `<app>/tests/` present? Translation tests?
- translations: `<app>/locale/` with catalogues? Which languages? A project
  tool like `tools/translate.py`?

## 1a · Install into the dev instance

Once the app module with `apps.py` exists:

1. Install it editable into the venv, **without dependencies**, so pip does
   not change Alliance Auth, Django or corptools of the dev instance:

   ```bash
   <python> -m pip install -q --no-deps -e <repo>
   ```

   If a dependency from `pyproject.toml` is then missing from the venv, name
   it to the user instead of installing it.
2. Add `'<app>',` at the end of `INSTALLED_APPS += [...]` in the local
   settings file (Edit tool, in the style of the existing entries). If it is
   already there, do nothing.
3. Check: `<python> manage.py check` from the AA instance must pass.
4. From the AA instance:

   ```bash
   <python> manage.py migrate --plan
   ```

   If the plan holds only migrations of the new app (or none), run
   `migrate` and `collectstatic --noinput`. If it holds migrations of
   **other** apps (Alliance Auth, corptools, sister apps), stop and show the
   plan - that was not part of the setup. With an irreplaceable database,
   ask before `migrate` even for the new app's migrations.

Read the output through the same DEBUG/INFO filter `aa-test` uses and report
only the last lines.

## 2 · CLAUDE.md

Start from `templates/CLAUDE.md`. Missing file: fill in `<project>` and
`<app>`, delete placeholder sections that do not apply (no translations, a
disposable database). Existing file: add only the missing sections, in the
tone of the file.

The **`## Release`** block is what `/commit` and `/push` read - keep exactly
its shape and its keys. Commands in it run from the AA instance; a bare
`python` stands for the Python from `## AA environment`, so the block holds
no machine paths and can be committed.

## 3 · CHANGELOG.md

Missing: copy `templates/CHANGELOG.md`. Existing without `[Unreleased]`: add
that section at the top, touch nothing else.

## 4 · docs/HANDOVER.md

Missing: copy `templates/HANDOVER.md` and fill in what step 1 found.

## 5 · Separate the translation tests

Only if there are catalogues. Every test class that checks catalogues
(translated entries, compiled `.mo`, jargon in `.po`) gets
`@tag("translations")` (`from django.test import tag`) and a short comment on
why: between commits the catalogues describe the last commit, not the code.
Tests that only check code (e.g. the shape of a glossary) stay untagged.

Check that both work:

- `aa-test <app> --exclude-tag translations`: no translation test runs
- `aa-test <app> --tag translations`: only translation tests run

Compare the test counts, do not just read "OK".

## 6 · Session start (optional)

Claude Code loads a project's `CLAUDE.md` only when the session starts in
the repo. If the user starts sessions somewhere else (a collection folder,
the home directory) and uses auto-memory, offer a `project` memory: where the
repo lives, its remote, and "read `CLAUDE.md` and `docs/HANDOVER.md` before
working". Otherwise skip.

## 7 · Wrap-up

Do not commit. Summarise briefly:

- files created or extended
- `## Release` verbatim
- test counts with and without translation tests
- installed and listed in the settings (or why not), result of
  `manage.py check`, `migrate` and `collectstatic`
- what is open, e.g. no tests, no version

Then: "Take it over with `/commit`."
