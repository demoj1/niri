# Maintainer: David Runge <dvzrv@archlinux.org>
# Maintainer: Caleb Maclennan <caleb@alerque.com>
# Maintainer: Robin Candau <antiz@archlinux.org>

pkgname=niri
pkgver=26.04
pkgrel=3
pkgdesc="A scrollable-tiling Wayland compositor"
arch=(x86_64)
url="https://github.com/niri-wm/niri"
license=(GPL-3.0-or-later)
depends=(
  cairo
  glib2
  glibc
  libdisplay-info
  libgcc
  libinput
  libpipewire
  libxkbcommon
  mesa
  pango
  pixman
  seatd
  systemd-libs
  xdg-desktop-portal-impl
)
makedepends=(
  clang
  git
  rust
)
optdepends=(
  'alacritty: a suggested GPU-accelerated terminal emulator'
  'bash: for niri-session script'
  'fuzzel: a suggested Wayland application launcher'
  'mako: a suggested Wayland notification daemon'
  'org.freedesktop.secrets: for apps to rely on secrets portal'
  'swaybg: a suggested Wayland wallpaper tool'
  'swaylock: a suggested Wayland screen locker'
  'waybar: a suggested Wayland customizable desktop bar'
  'xwayland-satellite: for running X11 apps in XWayland'
  'xdg-desktop-portal-gtk: a suggested XDG desktop portal'
  'xdg-desktop-portal-gnome: a XDG desktop portal required for screencasting'
)
provides=(wayland-compositor)
source=($pkgname-$pkgver.tar.gz::$url/archive/refs/tags/v$pkgver.tar.gz
        0001-Don-t-send-frame-callbacks-while-monitors-are-powere.patch
        0002-layout-Draw-active-decorations-only-on-the-focused-w.patch
        0003-overview-Add-focus-ring-and-shadow-overrides.patch)
sha512sums=('ae30446fbf529155907726cf2cba7402bcfc5be2032370d66f9f980efb257f29353b47d495f1218b81ac32c643f15bbb74834dc35882849a4479021b4b9fa97f'
            '636a74ab8e681f114f27bf5ec3c5fc135f08c91c02a722922a1178a6f6439da0d766e6f8e391f39ce1d8a8a257f1077ccd02fc8fd51b2c28b9430203b48a3c25'
            '6f3f3f8b205c78a1c66dfae9e5f2d60bb505e079f63f52a72e1ff2efce50097b33ee8ec2598d738d2545be4e154642a1a3b85a7d5b895a4347d9afd19cd3f786'
            'd608a3619d4e46d289fbd1a140148b514ae052578f30277b40eee99be6fba12ab33577bde773afbb20fae12dc1aeb8c07f0d51f98aa2bd1c6fac425dc20e34ac')
b2sums=('65fc4df85ab40c1a7a82900a9812c542a8a00467a0f1742f2dede50c0e323bd8957149e91b181a6c71ad077735f6a4db53451bbad0deb768f7d4840ebe798a20'
        'c327c2e2de0b0031667cf9fda3b97221dbda279d11ec259fbf863bf4692a51f782545e563943f56de9cf9dcd13185fabdfe346118d2723b9589e4290c28e7ff3'
        'b1f8a6e3616d69bbf69162d06356057cf57f482aedc41ae67baf5212951e43cea633996ace9f0dc10f5dbee5e599978f3e6468860e2c956d8fbacca64d98f62a'
        'de67078ee37d0abe5cc580ab12a16c28194c9bd11eeba2108bf93fbc620c55c8d56129d4640e87198fd8c04a99e94aa7228df5ee7a1a7d84316b06ad254e310c')

prepare() {
  cd $pkgname-$pkgver
  for p in ../000*.patch; do
    patch -Np1 -i "$p"
  done
  cargo fetch --locked --target "$(rustc --print host-tuple)"
}

build() {
  cd $pkgname-$pkgver
  export NIRI_BUILD_COMMIT="$(zcat ../$pkgname-$pkgver.tar.gz | git get-tar-commit-id | cut -c1-7)"
  CFLAGS+=(' -ffat-lto-objects')
  cargo build --frozen --release --features default

  # generate shell completions
  for shell in bash fish zsh; do
    cargo run --frozen --release --bin niri -- \
      completions "$shell" > "$shell-completions"
  done
}

check() {
  cd "$pkgname-$pkgver"
  export XDG_RUNTIME_DIR="$(mktemp -d)"
  export RAYON_NUM_THREADS=1  # required so we can build in environments with _many_ threads
  cargo test --all --exclude niri-visual-tests --frozen
}

package() {
  cd $pkgname-$pkgver
  install -vDm 755 {target/release/$pkgname,resources/$pkgname-session} -t "$pkgdir/usr/bin/"
  install -vDm 644 resources/$pkgname{.service,-shutdown.target} -t "$pkgdir/usr/lib/systemd/user/"
  install -vDm 644 resources/$pkgname.desktop -t "$pkgdir/usr/share/wayland-sessions/"
  install -vDm 644 resources/$pkgname-portals.conf -t "$pkgdir/usr/share/xdg-desktop-portal/"
  install -vDm 644 resources/default-config.kdl README.md -t "$pkgdir/usr/share/doc/$pkgname/"
  # shell auto-completions
  install -vDm 644 bash-completions "$pkgdir/usr/share/bash-completion/completions/niri"
  install -vDm 644 fish-completions "$pkgdir/usr/share/fish/vendor_completions.d/niri.fish"
  install -vDm 644 zsh-completions "$pkgdir/usr/share/zsh/site-functions/_niri"

}
