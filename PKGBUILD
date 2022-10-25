# Maintainer: Pierre-Loup A. Griffais <pgriffais@valvesoftware.com>

pkgname=gamescope
_srctag=3.11.33-beta1
pkgver=${_srctag//-/.}.snapshot$(date +%Y%m%d.%H%M)
pkgrel=4
pkgdesc="gaming shell based on Xwayland, powered by Vulkan and DRM. with fixes from Samsagax for screen orientation"
arch=(x86_64)
url="https://github.com/Plagman/gamescope"
license=('MIT')
depends=('xorg-xwayland-jupiter' 'libxres' 'xcb-util-errors' 'freerdp' 'xcb-util-wm' 'libxcomposite' 'pixman' 'libinput' 'seatd' 'pipewire')
makedepends=(git meson wayland-protocols ninja glslang vulkan-headers)
source=("gamescope-session"
        "gamescope-wayland.desktop"
        "gamescope-mimeapps.list"
        "steam_http_loader.desktop"
        "steam-http-loader"
        "git+https://github.com/theVakhovskeIsTaken/gamescope.git"
        "git+https://gitlab.freedesktop.org/wlroots/wlroots.git"
        "git+https://gitlab.freedesktop.org/emersion/libliftoff.git"
        # FIXME Upstream gamescope is just selecting master branch at build time, so we are arbitrarily snapshotting a
        #       revision when bumping the version here such that the build is reproducible.
        "git+https://github.com/nothings/stb.git#commit=af1a5bc352164740c1cc1354942b1c6b72eacb8a")
sha256sums=('SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP')

install=gamescope.install

prepare() {
	cd "$pkgname"

	# git submodules
	git submodule init
	git config submodule.subprojects/wlroots.url "$srcdir/wlroots"
	git config submodule.subprojects/libliftoff.url "$srcdir/libliftoff"
	git submodule update

	# meson subprojects
	rm -rf subprojects/stb
	git clone "$srcdir/stb" subprojects/stb
	cp -av subprojects/packagefiles/stb/* subprojects/stb/ # patch from the .wrap we elided
	sed -i 's/vec4(outputValue, 0)/vec4(outputValue, 1.0)/' src/shaders/cs_composite_blit.comp # Intel Xe gfx -> mouse-blanking workaround
}

build() {
	cd "$pkgname"

	rm -rf build
	mkdir build
	cd build
	arch-meson --buildtype release --prefix /usr ..
	ninja
}

package() {
	install -D -m 755 gamescope-session "$pkgdir"/usr/bin/gamescope-session
	install -D -m 644 gamescope-wayland.desktop "$pkgdir"/usr/share/wayland-sessions/gamescope-wayland.desktop

	# url handling
	install -D -m 644 steam_http_loader.desktop "$pkgdir"/usr/share/applications/steam_http_loader.desktop
	install -D -m 644 gamescope-mimeapps.list "$pkgdir"/usr/share/applications/gamescope-mimeapps.list
	install -D -m 755 steam-http-loader "$pkgdir"/usr/bin/steam-http-loader

	cd "$pkgname/build"

	DESTDIR="$pkgdir" meson install --skip-subprojects

	rm -rf "$pkgdir"/usr/include
	rm -rf "$pkgdir"/usr/lib/libwlroots*
	rm -rf "$pkgdir"/usr/lib/pkgconfig
}
