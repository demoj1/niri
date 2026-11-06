# niri 26.04 for Arch, with patches

| Patch | What it does |
|---|---|
| `0001` monitors-off frame callbacks | Fixes [niri-wm/niri#3295](https://github.com/niri-wm/niri/issues/3295): VRAM leak while monitors are powered off that kills X11 windows (Emacs, etc.) |
| `0002` one active ring in overview | In the overview only the focused workspace draws the active focus ring, so you can see which window is focused |
| `0003` overview focus-ring/shadow | New `focus-ring {}` and `shadow {}` inside `overview {}` to style windows differently in the overview |

Code lives on branch [`dmr/overview-decorations`](https://github.com/demoj1/niri/tree/dmr/overview-decorations) (one commit per patch, on top of v26.04).

## 0001: VRAM leak with monitors off

With monitors powered off, niri keeps sending frame callbacks without vblank throttling, X11 clients (via xwayland-satellite) pile up buffers until VRAM runs out, and their windows die (`Failed to allocate NVKMS memory for GEM object`).

The fix is one `if` in `Niri::redraw`. Tested overnight on NVIDIA: VRAM stays flat, clients resume rendering after power-on.

## 0003: overview decorations

Same options as the `focus-ring` and `shadow` window rules. Applied on top of `layout`, window rules still win. Sizes are zoomed with the workspace: `width 8` at `zoom 0.5` is 4 px on screen.

```kdl
overview {
    zoom 0.5
    focus-ring {
        width 8
        active-gradient from="#ff00ff" to="#ffff00" angle=45
    }
    shadow {
        color "#ff00ff60"
    }
}
```

> [!NOTE]
> DankMaterialShell's *Launcher → Overview → Overlay* grabs exclusive keyboard focus in the overview, so no window is focused and every ring is drawn inactive. Turn it off if you want to see the focused window.

## Install

```sh
git clone -b arch https://github.com/demoj1/niri.git niri-pkg
cd niri-pkg
makepkg -si
```

Then add `niri` to `IgnorePkg` in `/etc/pacman.conf`, otherwise the next `pacman -Syu` replaces it with the unpatched official package.

PKGBUILD is the official Arch one plus the patches applied in `prepare()`.
