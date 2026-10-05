---
name: push
description: >
  Publishes the committed state of an Alliance Auth / Django app: bump the
  version (default +0.0.1), split the CHANGELOG onto it, update
  docs/HANDOVER.md for a fresh session, release commit, then push - without
  asking when nothing stood out, otherwise ask first. Expects the work to be
  committed with /commit. Reads everything app-specific from "## Release" in
  the app's CLAUDE.md and everything machine-specific from
  "## AA environment". Only on an explicit /push.
disable-model-invocation: true
---

# Push

One or more committed states become a version in the remote, plus a handover
a new session can continue from without prior knowledge. No tests here -
they ran at `/commit`.

Talk to the user in their language; files and commit messages stay English.

Setup as in `/commit`: `## AA environment` for the machine (usually in
`~/.claude/CLAUDE.md`, possibly overridden in `CLAUDE.local.md`), only the
`## Release` section of the project's `CLAUDE.md` for the app. If either is
missing, stop and suggest `/project-setup`. Run every command - git
included - the way `Shell` says.

## 0 · Check the starting point

- `git status -s`: if uncommitted code changes are lying around, stop, name
  them (which files, briefly what) and settle both in one question: should
  they be committed first (with `/commit`), and should the push happen
  anyway? A release commit only takes version, CHANGELOG and HANDOVER along.
- `git log --oneline @{u}..HEAD`: the commits that go out. If the list is
  empty and there is nothing to version, stop and say so.

## 1 · Bump the version

The file named by `Version file`.

- If the user names a version ("make it 0.4.0"), set exactly that.
- Otherwise check whether it was already bumped since the last push
  (`git diff @{u} -- <file>`). If not: patch level +1 (0.3.7 → 0.3.8).

## 2 · Split the CHANGELOG

Move the contents of the section named by `Changelog section` into a new
section `## [<version>] - <today's date>` below it; the running section stays
behind, empty. Do not reword anything, only sort it in. If the section is
empty, ask whether to really publish without entries.

## 3 · Write the handover

Update `docs/HANDOVER.md` so that a new Claude session can continue right
away:

- version and date of this state
- what this version added (briefly, pointing to the CHANGELOG)
- open points, known bugs, sensible next steps
- the user's decisions from this session that are not in the code
- state of the dev instance (unapplied migrations, seeded data)
- traps this session lost time on

Rewrite existing sections and delete what is outdated instead of appending -
the file describes a moment, not a history (that is the CHANGELOG's job).
Lasting rules belong in `CLAUDE.md`, not here.

## 4 · Release commit

Stage version, CHANGELOG and HANDOVER by path. Write the message with the
Write tool to `<repo>/.git/CLAUDE_COMMIT_MSG`, then
`git commit -F .git/CLAUDE_COMMIT_MSG` - never through a heredoc. Subject
`Release <version>`, ending with the attribution line from the session's
system context, if there is one.

## 5 · Push

First compare with the remote: `git fetch`, then
`git rev-list --count HEAD..@{u}` (must be 0) and `git ls-remote origin
<branch>` against `git rev-parse @{u}` (must match - a release can have been
left behind locally).

**Push without asking** when all of this holds:

- Step 0 found no uncommitted code changes and no empty commit list.
- Step 2 did not have to ask (the running section had entries).
- The release commit went through.
- The remote is not ahead of `@{u}`, and both comparisons above match.
- Nothing else in this run raised a question, an error or a surprise.

Then `git push` - never `--force` - and say in the summary that it was
pushed without asking.

**Otherwise ask first** and wait for a yes, naming what stood out:

> Push `<branch>` to `<remote>`: version <old> → <new>, <n> commits
> (<subjects, short>). Stood out: <what>. Push anyway?

If the push fails (the remote is ahead), do not rebase or merge - show the
situation and ask.

## 6 · Summary

Short, about six lines at most:

- version old → new, branch, remote, hash of the release commit
- pushed commits (count, subjects)
- what the CHANGELOG lists under the version (keywords)
- what HANDOVER.md names as next steps
- note: start a new session for the next topic
