---
name: commit
description: >
  Commits the current state of an Alliance Auth / Django app once it checks
  out: test suite without translation tests, checks, translations if needed,
  translation tests, then a local commit. No version bump, no push - that is
  /push. Reads everything app-specific from the "## Release" block of the
  app's CLAUDE.md and everything machine-specific from "## AA environment".
  Only on an explicit /commit.
disable-model-invocation: true
---

# Commit

One checked local commit of the current state. **No version bump, no
CHANGELOG split, no push** - all of that belongs to `/push`.

Talk to the user in their language; commit messages stay English.

## 0 · Setup

Two blocks drive this skill:

- **`## AA environment`** - the machine: where repos and the AA instance
  live, which Python, how to run a command (natively or through WSL), how to
  reach files. Usually in `~/.claude/CLAUDE.md` and therefore already in
  context; a project's `CLAUDE.local.md` may override it.
- **`## Release`** in the project's `CLAUDE.md` - the app: commands for
  tests, checks and translations. If the file is not in context yet, read
  only that section (`grep -n "^## Release" -A 25 CLAUDE.md`).

If either block is missing, stop and suggest `/project-setup` - do not guess.

Conventions in `## Release`: commands run from the AA instance, and a bare
`python` means the Python from `## AA environment`.

Run every command - git included - the way `Shell` in `## AA environment`
says. Read test output by its last lines (`| tail -25`) unless something
failed.

## 1 · Suite without translation tests, then checks

Run the lines `Suite without translation tests` and `Checks` from
`## Release`. If a run hangs with `--parallel`, repeat it without the flag -
a failing subtest blocks the parallel runner.

**On failure:** fix the cause (do not weaken the test), run again, tell the
user briefly what was broken and what changed, and **ask whether to commit
now**. Continue only on yes.

## 2 · Translations, if needed

Needed when translatable texts are new or changed since the last commit
(look through `git diff HEAD -- '*.py' '*.html'` for `gettext`, `translate`,
`_(`). Otherwise skip the step and say so. If `Translations` says "none",
the step does not apply.

Follow the `Translations` line of `## Release` exactly (and the section it
points to). In general:

- EVE jargon stays English.
- Never take a `#, fuzzy` suggestion unread - msgfmt does not compile fuzzy
  entries, and gettext's guess is usually wrong.
- Fill `.po` files with the Edit tool, not a script - unless the project has
  its own tool for it, then never by hand.
- List every newly translated text in the summary so the user can review it.

## 3 · Translation tests only

Run the line `Translation tests` (skip it when the tool from step 2 already
ran them, or there are none). Failure as in step 1: fix, report, ask.

## 4 · Commit

1. Look at `git status -s`. Does the list match the work? Do not take
   unknown untracked files along unasked - ask.
2. Stage files by path (`git add <paths>`), no blind `git add -A`.
3. Write the commit message with the **Write tool** to
   `<repo>/.git/CLAUDE_COMMIT_MSG` (through the file path from
   `## AA environment`), then `git commit -F .git/CLAUDE_COMMIT_MSG`.
   **Never through a heredoc** - apostrophes and brackets have cut a message
   off mid-sentence before.
4. Message: English, one subject line in the style of `git log --oneline -5`,
   below it briefly the why. End with the attribution line from the current
   session's system context, if there is one.
5. Check `git log -1 --format="%h %s"`.

## 5 · Summary

Short, about six lines at most:

- commit hash and subject
- tests: X passed (without translation tests), checks ok, translation tests
  Y passed / not applicable
- translated: yes (which texts) / not needed
- what was fixed along the way, if anything
- note: version and push follow with `/push`
