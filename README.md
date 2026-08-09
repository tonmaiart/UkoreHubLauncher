# UkoreHubLauncher

The `UkoreHubLauncher.exe` launcher for [UkoreHub](https://github.com/tonmaiart/UkoreHub)
(a.k.a. "UkoreHubRelease" — the app repo artists actually run), split into
its own repo so an ordinary app-code release can never again try to
overwrite the exe file that's busy running it. See
`developer/bug-history/2026-08-08-self-update-locked-own-exe.md` and
`developer/README.md`'s "Repo split" section in the UkoreHubDev repo for
the full history/reasoning.

## Layout

Artists get a folder that looks like this after running `UkoreHubLauncher.exe`
once:

```
UkoreHubLauncher.exe        <- this repo's own working tree, self-updates rarely
app/                <- clone of the UkoreHub app repo, gitignored here,
                        bootstrapped/updated on every launch
  launcher.py
  core/  interface/  plugins/  data/  cache/  projects/  ...
```

Double-click `UkoreHubLauncher.exe` — same as before the split. It brings this
repo up to date first (rare — only when an admin rebuilds/republishes the
exe), then bootstraps or updates the nested `app/` clone (frequent —
every ordinary app release), then checks Python/git-lfs, handles GitHub
login, and spawns `app/launcher.py`.

## Files

All the build-time inputs live under `launcher_build/` so the repo root
only has to carry the tracked `UkoreHubLauncher.exe` binary itself:

- `launcher_build/exe_entry.py` — the tiny script PyInstaller compiles
  into `UkoreHubLauncher.exe`. Hands off to `updater.py`'s `main()`.
- `launcher_build/updater.py` — the actual pre-launch logic: git/Python
  prerequisite checks, self-update of this repo, bootstrap/update of
  `app/`, GitHub device-flow login (tkinter UI), then spawns
  `app/launcher.py`. See its own module docstring for the full breakdown,
  including why `core/exceptions.py`/`core/models.py`/`core/paths.py`/
  `core/theme.py`/`core/store.py`/`core/github/` here are **vendored
  copies** of the app repo's identically-named files rather than an
  import — a separate repo has no way to import the app repo's package
  directly, and these are all confirmed stdlib-only. A change to the real
  ones (OAuth flow, token storage, config schema) needs to be manually
  mirrored here if it should also apply to this pre-launch login screen.
- `launcher_build/build_exe.py` — admin-only: run this to (re)build
  `UkoreHubLauncher.exe` at this repo's root after rebranding `icon.ico` or
  changing `exe_entry.py`/`updater.py` themselves. Installs
  `pyinstaller`/`keyring` into the current environment if missing.
  `--icon`/`--name` CLI args, defaults to `icon.ico`/"UkoreHub".
- `launcher_build/icon.ico` — the icon baked into `UkoreHubLauncher.exe`. Swap
  this file and rerun `build_exe.py` to rebrand.

`UkoreHubLauncher.exe` itself is git-tracked at this repo's root (deliberately not
gitignored) — a shared studio-wide binary rebuilt and recommitted whenever
an admin runs `build_exe.py`, same pattern the app repo uses for
`data/thumbnails/`.

## Publishing a new build

```bash
python launcher_build/build_exe.py
git add UkoreHubLauncher.exe
git commit -m "..."
git push
```

Already-installed artists pick it up automatically the next time they
launch `UkoreHubLauncher.exe` (the rare self-update step above).
