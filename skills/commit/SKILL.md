---
name: commit
description: >
  Commits the current state of an Alliance Auth / Django app once it checks
  out: test suite without translation tests, checks, translations if needed,
  translation tests, version bump (default +0.0.1) with the CHANGELOG split
  onto it, then a local commit. No push - that is /push. Reads everything app-specific from the "## Release" block of the
  app's CLAUDE.md and everything machine-specific from "## AA environment".
  Only on an explicit /commit.
disable-model-invocation: true
---

# Commit

One checked local commit of the current state, and every commit is a new
version: it bumps the version and moves the running CHANGELOG entries onto
it. **No push, no handover, no tag** - those belong to `/push`.

Talk to the user in their language; commit messages stay English.

## 0 · Setup

Two blocks drive this skill:

- **`## AA environment`** - the machine: where repos and the AA instance
  live, which Python, how to run a command (natively or through WSL), how to
  reach files. Usually in `~/.claude/CLAUDE.md` and therefore already in
  context; a project's `CLAUDE.local.md` may override it.
- **`## Release`** in the project's `CLAUDE.md` - the app: commands for
  version file, changelog section, tests, checks and translations. If the file is not in context yet, read
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

## 4 · Version and CHANGELOG

The file named by `Version file`:

- If the user names a version ("make it 0.4.0"), set exactly that.
- Otherwise check whether it was already changed since the last commit
  (`git diff HEAD -- <file>`). If not: patch level +1 (0.3.7 → 0.3.8).

Then move the contents of the section named by `Changelog section` into a
new section `## [<version>] - <today's date>` below it; the running section
stays behind, empty. Do not reword anything, only sort it in. If the section
is empty, propose entries from the diff and ask before writing them.

Both files go into the commit below.

## 5 · Commit

1. Look at `git status -s`. Does the list match the work (plus version file
   and CHANGELOG)? Do not take unknown untracked files along unasked - ask.
2. Stage files by path (`git add <paths>`), no blind `git add -A`.
3. Write the commit message with the **Write tool** to
   `<repo>/.git/CLAUDE_COMMIT_MSG` (through the file path from
   `## AA environment`), then `git commit -F .git/CLAUDE_COMMIT_MSG &&
   rm .git/CLAUDE_COMMIT_MSG`.
   **Never through a heredoc** - apostrophes and brackets have cut a message
   off mid-sentence before. Run the commit only after the Write succeeded, in
   a later message - never in the same one. If the Write fails ("File has not
   been read yet": a leftover from an aborted run), read the file, write
   again, then commit; otherwise git commits the stale message.
4. Message: English, one subject line in the style of `git log --oneline -5`,
   below it briefly the why. End with the attribution line from the current
   session's system context, if there is one.
5. Check `git log -1 --format="%h %s"`.

## 6 · Summary

Short, about six lines at most:

- commit hash and subject, version old → new
- tests: X passed (without translation tests), checks ok, translation tests
  Y passed / not applicable
- translated: yes (which texts) / not needed
- what was fixed along the way, if anything
- note: handover, tag and push follow with `/push`
