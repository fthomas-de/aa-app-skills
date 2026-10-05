# Working on <project>

<One sentence: what the app does, for which Alliance Auth version.>
`README.md` says what the app does; this file says how to work on it;
`docs/HANDOVER.md` says where the work currently stands.

## Where things are

| | |
|---|---|
| This app | `<app>/` in this repo |
| Tests | `<app>/tests/` |
| Translations | `<app>/locale/` |
| AA instance, Python, database | see `## AA environment` (machine-specific, usually in `~/.claude/CLAUDE.md`) |

Everything runs from the Alliance Auth instance, not from the app directory.

## Commands

A bare `python` below means the Python from `## AA environment`.

```bash
aa-test <app>                         # whole suite, quiet
aa-test <app>.tests.<module>          # one module
aa-test <app> --fresh                 # rebuild the test database first
python manage.py makemigrations <app>
python manage.py collectstatic --noinput
```

`aa-test` wraps `manage.py test` and drops the DEBUG/INFO lines Alliance
Auth's console logging adds, nothing else - a real traceback, `WARNING` or
`ERROR` still comes through. It keeps the test database (`--keepdb`);
`--fresh` rebuilds it, and the suite at `/commit` always runs fresh. No
`--parallel`: with `--keepdb` it saves little, does not migrate its clone
databases and hangs on a failing subtest.

## Release

Read by the skills `/commit` and `/push`; the same shape in every app.
Commands run from the AA instance. Translation tests carry
`@tag("translations")`: between commits the catalogues describe the last
commit, so the suite leaves them out.

- App: `<app>`
- Version file: `<app>/__init__.py`
- Changelog section: `[Unreleased]`
- Tests while working: `aa-test <app>.tests.<module>`
- Suite without translation tests: `aa-test <app> --fresh --exclude-tag translations`
- Checks: `python manage.py makemigrations <app> --check --dry-run`
- Translations: <makemessages, fill every `.po`, `msgfmt --check`,
  compilemessages, `.mo` newer than `.po` - or "none yet">
- Translation tests: <`aa-test <app> --tag translations` - or "none yet">

## The dev database

<Keep this section if `## AA environment` marks the database irreplaceable,
e.g. because it holds ESI data that cannot be fetched again.>

- Never delete with a range filter on a model other apps hang off - a
  cascade on `EveCorporationInfo` takes corptools' wallet journal with it.
- Address rows by explicit id lists, and write the previous values out first.
- Foreign apps (Alliance Auth, corptools, ...): read their models, never
  change their schema or their rows beyond what a seed of our own created.

## Code

Comments say **why**, not what. A comment that restates the line is noise; a
comment that names the fault the line prevents keeps it from coming back.

English everywhere in the code, including comments and docstrings.

Templates use the Alliance Auth standard style unchanged: its Bootstrap 5
classes, its bundles, `{% load sri %}` with `{% sri_static %}` for our own
scripts. No custom theme. Prefer an Alliance Auth or corptools pattern over
inventing one.

JavaScript and CSS live in `<app>/static/<app>/`, never inline in a
template - `sri_static` cannot hash inline code, and `collectstatic` has to
see the file. Run `collectstatic` after every change to them.

## Tests

Every change gets a test, and every new test gets checked against the broken
version: put the fault back, run the test, confirm it fails, restore the
file. A test that passes against the broken code tests nothing.

Test users get no password, and tests log in with `force_login`. Django
hashes a password with PBKDF2 at about 0.2 seconds a user.

## Translations

<Languages, plural forms, project tool if any. Then:>

- EVE jargon stays English in every language: Corporation, Alliance,
  Character, Main, ...
- A `#, fuzzy` entry is not compiled by msgfmt, and gettext's guess is
  usually wrong. Read every one.
- After `compilemessages`, check that the `.mo` files are newer than their
  `.po` - a command chained behind a non-zero exit never runs, silently.
- Fill `.po` files with the Edit tool; `msgfmt --check -o /dev/null <file>`
  after any bulk edit.
- `makemessages` without `--no-location`: the flag strips every `#:` line
  from all catalogues at once.

## Committing

Commits and pushes only go through the skills `/commit` and `/push`, never
unasked; both read `## Release` above. While working on a feature, run only
the affected test modules - the full suite runs at `/commit`.

`CHANGELOG.md` is written along with every change, unasked, under
`[Unreleased]`. An entry says what was wrong and why the fix is the fix.

## Editing files

Write patch scripts with the Write tool and run them by path - never build
them with shell heredocs: an apostrophe or a backtick gets executed by bash
before Python sees the text. Every patch script asserts its anchor occurs
exactly once before it replaces anything. For a small, unambiguous change the
Edit tool is one step instead of two - use it.

## Keeping a session cheap

Every request re-sends the whole conversation, so a long session costs
several times what the same work costs in a fresh one. One session per
topic; start a new one when the subject changes.
