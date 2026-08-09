# UkoreHubLauncherDev

Dev repo for `UkoreHubLauncher.exe` — see [README.md](README.md) for the
full repo-split rationale and layout. This file only covers rules that
aren't obvious from reading the code.

## Publishing — `developer/commit-release-fast.ps1` always rebuilds the exe first

It runs `python launcher_build/build_exe.py` before `git add -A`, so the
`UkoreHubLauncher.exe` that gets committed/pushed (and mirrored to the
release repo) is never stale relative to `exe_entry.py`/`updater.py`. A
failed build stops the script before anything is staged — don't rebuild
manually before running this script, it's redundant.

## The release repo never carries a `.gitignore`

`developer/commit-main.ps1` mirrors this repo's `main` onto the
`UkoreHubLauncher` release repo, stripping `developer/`, `.claude/`,
`launcher_build/`, **and this repo's own `.gitignore`** — artists only
ever pull that repo and nothing there runs `git add -A`, so there's
nothing for a `.gitignore` to protect.
