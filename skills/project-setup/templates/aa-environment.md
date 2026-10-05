## AA environment

Read by the skills `/commit`, `/push` and `/project-setup` (aa-app-skills).
Describes this machine; each app's own `## Release` block describes the app.

- Repos: `~/aa-dev/working/<project>`
- AA instance: `~/aa-dev/working/myauth` (has `manage.py`; app commands run from here)
- Python: `~/aa-dev/venv/bin/python`
- Local settings: `myauth/settings/local.py` (relative to the AA instance)
- Database: `aa_dev` - irreplaceable: never delete rows unasked, never
  migrate other apps without a yes
- Test runner: `aa-test` (from `project-setup/scripts`, in `~/bin`)
- Shell: native
- Files: the repo path itself

<!--
Variant for Claude Code on Windows with the instance in WSL - replace the
last two lines with:

- Shell: `wsl.exe -d Ubuntu-22.04 -e bash -lc 'cd <dir> && <command>'` -
  git and every project command run inside WSL; never set `safe.directory`
  (git on the Windows path reports "dubious ownership"). Shell variables do
  not survive the bridge: write commands out.
- Files: `\\wsl.localhost\Ubuntu-22.04\home\<user>\...` for Read/Write/Edit

A disposable database instead: "- Database: `<name>` - disposable".
-->
