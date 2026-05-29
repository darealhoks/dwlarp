# dwlarp

A complete, statically-configured Wayland desktop built on
[dwl](https://codeberg.org/dwl/dwl) 0.8 — the suckless `dwm`-for-Wayland
compositor — plus [**wisp**](https://github.com/darealhoks/wisp), a single C
daemon that provides the bar, status, HUD, OSD, app menu, and desktop
notifications.

Configuration is compile-time only (the suckless model): there are no runtime
config files and no theme overlay. A single per-host `config.h` is the source
of truth, and a POSIX-sh installer targets **Void Linux**.

## What's in the box

| component | role |
|-----------|------|
| `dwl/` | the compositor (dwl 0.8), built against the active host config |
| **wisp** | widget daemon — bar, status, HUD, OSD, menu, notifications, lock (separate repo) |
| `mullvad-menu/` | Mullvad VPN picker/control |
| `scripts/` | session launcher, autostart/autolayout, app launcher, screenshot, pomodoro, mullvad helpers |
| `hosts/<name>/config.h` | the full per-host configuration |

wisp replaces the entire old per-component stack (`dwlb`, `dwlb-status`,
`dwlb-leftstatus`, `ws-hud`, `mako`, `dmenu-wl`).

## Install

```sh
git clone https://github.com/darealhoks/dwlarp.git
cd dwlarp
./install.sh
```

or straight from the network:

```sh
curl -fsSL https://raw.githubusercontent.com/darealhoks/dwlarp/main/install.sh | sh
```

The installer:

- picks the host config from the machine's hostname (override with
  `DWLARP_HOST=<name>`), falling back to `hosts/void`;
- builds and installs **dwl** (against the host config via the Makefile's
  `HOSTCONFIG` variable), **wisp**, and **mullvad-menu**;
- installs the scripts and substitutes install-time templates (`*.in`,
  `assets/*.in`).

**wisp** lives in its own repo. For a local checkout the installer uses a
sibling `../wisp` directory if present; otherwise (including the `curl | sh`
path) it clones [`darealhoks/wisp`](https://github.com/darealhoks/wisp)
automatically. Point it at an existing working copy with `WISP_SRC=/path/to/wisp`.

Rebuild after editing config with:

```sh
./install.sh -r        # --rebuild
```

**Requirements:** wlroots 0.19 (dwl 0.8). The installer uses the distro's
wlroots package when available, otherwise builds it from source into
`/usr/local`. Non-Void distros are detected but Void is the primary target.

## Configuration

Everything tunable lives in **`hosts/<host>/config.h`** — compositor colors,
fonts, keybindings, app commands, lock command, and Mullvad cadences, all as
`WS_*` macros. There is no inheritance between hosts and no generated root
`config.h`: each host file is complete and used directly.

- dwl includes it at compile time via `-DDWLARP_CONFIG="…"` (`dwl/config.h`
  does `#include DWLARP_CONFIG`).
- Install-time templates read values from it via `read_str` / `read_num`.
- Optional `hosts/<name>/pkgs.void` lists extra xbps packages for that host.

**Widget appearance** (bar/HUD/OSD/menu colors, sizes, the HUD button table)
is *not* here — it's wisp's own `config.h` in the wisp repo, since wisp is
statically configured too.

The canonical loop: edit `hosts/<host>/config.h` (or wisp's `config.h` for
widgets), then `./install.sh -r`. Don't introduce runtime config files.

## Session

`scripts/dwlarp` is the entry point; it launches:

```sh
dwl -s "$HOME/.local/bin/dwl-autostart & $HOME/.local/bin/wisp"
```

wisp then owns the bar (top), the hover-revealed HUD, the centered OSD slabs,
the `dmenu`-replacement menu, and `org.freedesktop.Notifications`. Volume,
brightness, and mic keys are bound directly to `wispctl` via the host config's
`WS_OSD_CMD`.

## License

`dwl/` is dwl, licensed GPL-3.0 (see its headers / upstream). The surrounding
dwlarp tooling has no separate license file yet.
