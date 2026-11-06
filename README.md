# niri 26.04 for Arch, patched for the monitors-off VRAM leak

Fixes [niri-wm/niri#3295](https://github.com/niri-wm/niri/issues/3295): with monitors powered off, niri keeps sending frame callbacks without vblank throttling, X11 clients (via xwayland-satellite) pile up buffers until VRAM runs out, and their windows die (`Failed to allocate NVKMS memory for GEM object`).

The fix is one `if` in `Niri::redraw` — see [the commit](https://github.com/demoj1/niri/commit/98278219) on branch `dmr/monitors-off-frame-callbacks`. Tested overnight on NVIDIA: VRAM stays flat, clients resume rendering after power-on.

## Install

```sh
git clone -b arch https://github.com/demoj1/niri.git niri-pkg
cd niri-pkg
makepkg -si
```

Then add `niri` to `IgnorePkg` in `/etc/pacman.conf`, otherwise the next `pacman -Syu` replaces it with the unpatched official package. Remove it once the fix lands upstream.

PKGBUILD is the official Arch one plus the patch in `prepare()`.
