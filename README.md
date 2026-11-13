# install-fitgirl

Install a FitGirl repack on Linux with one command: no installer window, no
installer music, and a Lutris entry at the end.

```sh
install-fitgirl "/path/to/Some Game [FitGirl Repack]" --games-dir /mnt/games
```

The script contains no game files and downloads nothing. It only automates an
installer you already have.

## What it does

1. Checks the repack's `.bin` files against `MD5/fitgirl-bins.md5`. Optional
   files you chose not to download are skipped.
2. Runs `setup.exe /SILENT` under GE-Proton (through `umu-run`) on a hidden
   `gamescope --backend headless` display, with Wine's audio drivers disabled
   for that run.
3. Watches the hidden display: prints progress, and if the installer shows an
   error dialog (for example `ISDone.dll` / `Unarc.dll returned an error code`),
   saves a screenshot of it and stops.
4. Closes the post-install programs that would otherwise wait for a click:
   the bundled verifier (QuickSFV) and the DirectX and Visual C++ redist
   installers. It then verifies every installed file against
   `_Redist/fitgirl.md5` itself.
5. Adds the game to Lutris, using the prefix the installer ran in.

## Requirements

- `umu-run` (umu-launcher)
- `gamescope` with the `headless` backend
- Python 3.8+ with `python-xlib` and `PyYAML`
- ImageMagick (`import`), used only for the error screenshot
- A GE-Proton build in `~/.local/share/Steam/compatibilitytools.d` or
  `~/.local/share/lutris/runners/wine`. If none is found, umu downloads the
  latest one.
- Lutris, unless you pass `--no-lutris`

On Arch-based systems:

```sh
sudo pacman -S umu-launcher gamescope python-xlib python-yaml imagemagick
```

## Install

```sh
curl -Lo ~/.local/bin/install-fitgirl https://raw.githubusercontent.com/norman-ags/install-fitgirl/main/install-fitgirl
chmod +x ~/.local/bin/install-fitgirl
```

## Options

| Option | Meaning |
|---|---|
| `--games-dir DIR` | Folder that holds your games. The game goes in `DIR/<name>`. Default `~/Games`, or `$FITGIRL_GAMES_DIR`. |
| `--dest DIR` | Exact install folder, instead of `--games-dir`. Must be empty or not exist. |
| `--name NAME` | Name for the folder and the Lutris entry. Default: the repack folder name without the `[...]` suffix. |
| `--exe PATH` | Game executable, relative to the install folder. Default: a guess (see limitations). |
| `--prefix DIR` | Wine prefix. Default `~/Games/lutris-prefixes/<slug>`, or `$FITGIRL_PREFIX_DIR/<slug>`. |
| `--proton DIR` | GE-Proton folder to use. Default: the newest one installed. |
| `--no-lutris` | Install and verify only. |
| `--skip-source-check` | Skip the MD5 check of the `.bin` files. |

Logs go to `~/.local/share/lutris/install-logs/<slug>/` (`setup.log` from the
installer, `wine.log` from Wine and gamescope).

## The bug this works around

If you start a FitGirl installer from a shell that has
`NoDefaultCurrentDirectoryInExePath=1` in its environment, it fails on the
first archive, either with a page fault inside `setup.tmp` or with
`Unarc.dll returned an error code: -12 ... failed CRC check`. Wine honours that
Windows variable, and it stops `unarc.dll` from finding its helper
decompressors in the current folder. Some terminals and coding-agent sandboxes
export it. Changing Wine versions does not help; unsetting the variable does.
The script always removes it.

## Limitations

- **Tested on one machine only.** CachyOS (Arch-based), GNOME on Wayland,
  x86_64, GE-Proton11-12, umu-launcher 1.4.3, gamescope 3.16.25. Other distros
  should work if the requirements are met, but nobody has tried.
- **Tested on few repacks.** FitGirl's installer template changes over time,
  and older or newer repacks may behave differently.
- **DirectX and Visual C++ redistributables are never installed.** The script
  kills their installers when they start. Proton normally covers them; if a
  game needs one, install it into the prefix yourself (for example with
  winetricks).
- **Every other installer checkbox stays at its default.** You cannot pick language
  packs or other optional components, and you cannot turn on "Limit installer
  to 2 GB of RAM". For language packs, the practical control is which
  `fg-optional-*.bin` files you downloaded.
- **Low-memory machines are not handled.** The 2 GB limit is the installer's
  answer for machines with 8 GB of RAM or less, and this script cannot set it.
- **The "open FitGirl's website" step is blocked by disabling Wine's browser
  helper. This is untested**; a browser tab may still open at the end.
- **The hosts-file step still runs** and fails harmlessly inside the prefix.
- **The game executable is guessed**: the largest `.exe` in the install root
  (else anywhere in the tree), ignoring uninstallers, crash handlers and
  redistributables. Games with a separate launcher may get the wrong one; pass
  `--exe`, or fix the path in Lutris.
- **Crack variants are not selected.** If a repack ships alternatives in a
  `NoDVD` folder, copying one in is up to you.
- **Lutris is edited directly** (`pga.db` and a YAML file), not through a
  Lutris API. A running Lutris needs a restart to show the game, and a Lutris
  update could change that format.
- **The Lutris entry has no runner version pinned**, so the game runs with your
  Lutris default Wine/Proton, in a prefix created by GE-Proton.
- **No uninstall and no resume.** A failed run leaves a partial install folder;
  delete it before retrying.
- **Proton only.** Plain Wine without umu is not supported.

## License

MIT
