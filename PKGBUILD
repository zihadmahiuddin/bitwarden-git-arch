# Maintainer: Alexander Epaneshnikov <alex19ep@archlinux.org>
# Contributor: libertylocked <libertylocked@disroot.org>

pkgname=bitwarden
pkgver=2025.2.1
pkgrel=1
_electronversion=34
pkgdesc='A secure and free password manager for all of your devices'
arch=('x86_64')
url='https://github.com/bitwarden/clients/tree/master/apps/desktop'
license=('GPL-3.0-only')
depends=("electron$_electronversion" 'libnotify' 'org.freedesktop.secrets' 'libxtst' 'libxss' 'libnss_nis')
makedepends=('git' 'npm' 'python' 'python-setuptools' 'node-gyp' 'nodejs-lts-iron' 'jq' 'rust' 'fdupes')
source=(bitwarden::git+https://github.com/bitwarden/clients.git#branch=km/ssh-authorize-remember
        messaging.main.ts.patch
        nativelib.patch
        native-messaging.main.ts.patch
        remove-argon2-browser.patch
        no-sourcemaps.patch
        remove-unnecessary-deps.patch
        ${pkgname}.sh
        ${pkgname}.desktop)
sha512sums=('SKIP'
            '759db11cae26b8228000c98eb7bd21d0a46c964a858d27655f8f09114f5f7cba856623c3cad07424ba360e74144d9c0c050ee7219f8fe530cd9059d9f937f023'
            '985a41cd851710d74ee55bfe9cb9000e8e2c29d8d6aab82f5d3c4072c12511d2fcb4484b2f8b608856b68d36aa6a495c05094e7e67dda357a937a8a4802da5e4'
            'f12482139463c6f471b49f789cd7f7e1748ecd06e169e5519edb29d762f456b9050871043336f98595b1cf15d99206bcad41f92a7145beb67c154e2a01e4c740'
            'e5b83c9f0f8669d37f31870856f10447126ebeca2a42b042830983fe6d26073473afbc24e63c1c3f8ae43df6b6799de49a5fd82fc11fc3fd8db6088cfc625cd0'
            '9bf10ac9b02c687058b8c587eff1a55b30447b4fb4ab8a55aae5ef4898189ccc5c27d372c996564b8ed6c1ffea39c72bdf241ae849d9490dc4e59e1d02efc9b8'
            '472ade92af912ff51a1c226663c9675eea4f1b4ee4b4feca50e153a676ed69053ad9dbb72b7a0dbcf6876bd72c449ecfadb34419cd7a2b3d1f0b07cfb60a8fa5'
            '09acb1f4a7fb04fda120eda79ee847f285a421bc5bf5c3d42c78767f01f2051984b928019707c7c59e48e87728ab45ee2a98ff7ccee6b4e14bfdff93cd1106f0'
            'fdc047aadc1cb947209d7ae999d8a7f5f10ae46bf71687080bc13bc3082cc5166bbbe88c8f506b78adb5d772f5366ec671b04a2f761e7d7f249ebe5726681e30')

prepare() {
	cd bitwarden/apps/desktop

	export npm_config_cache="$srcdir/npm_cache"
	export ELECTRON_SKIP_BINARY_DOWNLOAD=1

	# Remove unused postinstall script (electron-rebuild)
	sed -i '/"postinstall":/d' package.json

	# Patch build to make it work with system electron
	export SYSTEM_ELECTRON_VERSION=$(electron$_electronversion -v | sed 's/v//g')
	export ELECTRONVERSION=$_electronversion
	sed -i "s|@electronversion@|${ELECTRONVERSION}|" "$srcdir/bitwarden.sh"

	cd ../../
	# This patch is required to make "Start automatically on login" work
	patch -p1 -i "$srcdir/messaging.main.ts.patch"
	# This patch is required to make "browser integration" work
	patch -p1 -i "$srcdir/native-messaging.main.ts.patch"
	# rust build
	patch -p1 -i "$srcdir/nativelib.patch"
	# patches from opensuse
	patch -p1 -i "$srcdir/no-sourcemaps.patch"
	patch -p1 -i "$srcdir/remove-argon2-browser.patch"
	patch -p1 -i "$srcdir/remove-unnecessary-deps.patch"
	npm ci
	pushd apps/desktop/desktop_native
	cargo fetch --locked --target "$(rustc -vV | sed -n 's/host: //p')"
	popd
}

build() {
	cd bitwarden/apps/desktop
	electronDist=/usr/lib/electron$_electronversion
	electronVer=$(electron$_electronversion --version | tail -c +2)
	export npm_config_cache="$srcdir/npm_cache"
	export ELECTRON_SKIP_BINARY_DOWNLOAD=1
	export CXXFLAGS="$CXXFLAGS -Wno-error"
	pushd desktop_native
	cargo rustc --frozen --release --package desktop_napi --lib --crate-type cdylib
	cargo rustc --frozen --release --package desktop_proxy --bin desktop_proxy
	mv -v target/release/*.so napi/desktop_napi.node
	popd
	npm run build
	npm run clean:dist
	mv -v desktop_native/target/release/desktop_proxy build/desktop_proxy
	rm -fv ../../node_modules/argon2/build-tmp-napi-v3/node_gyp_bins/python3
	fdupes build
	pushd build
	#JS debug symbols (unusable)
	find -name '*.map' -type f -print -delete
	#Source code
	find -name '*.c' -type f -print -delete
	find -name '*.cpp' -type f -print -delete
	find -name '*.h' -type f -print -delete
	find -name '*.gyp' -type f -print -delete
	find -name '*.gypi' -type f -print -delete
	find -name '*.ts' -type f -print -delete
	find -name '*.cts' -type f -print -delete
	find -name build-tmp-napi-v3  -print0 |xargs -r0 -- rm -rvf --
	find -name src -print0 |xargs -r0 -- rm -rvf --
	find -name Makefile -type f -print -delete
	find -name 'Pipfile*' -type f -print -delete
	find -name '*.patch' -type f -print -delete
	#Temporary build files
	find -name '.deps' -print0 |xargs -r0 -- rm -rvf --
	find -name 'obj.target' -print0 |xargs -r0 -- rm -rvf --
	find -name 'vendor' -print0 |xargs -r0 -- rm -rvf --
	find -name '*package-lock.json' -type f -print -delete
	find -name '*.mk' -type f -print -delete
	find -name '*.Makefile' -type f -print -delete

	#Documentation
	find -name '*.md' -type f -print -delete
	find -name doc -print0 |xargs -r0 -- rm -rvf --
	find -name test -print0 |xargs -r0 -- rm -rvf --
	#Compile-time-only dependencies
	find -name nan -print0 |xargs -r0 -- rm -rvf --
	find -name node-addon-api -print0 |xargs -r0 -- rm -rvf --
	#Other trash
	find -name '*.yml' -type f -print -delete
	find -name '.npmignore' -type f -print -delete
	find -name '.gitignore' -type f -print -delete

	#Fix file mode
	find . -type f -exec chmod 644 {} \;
	find . -name '*.node' -exec chmod 755 {} \;
	chmod 755 desktop_proxy

	# Remove empty directories
	find . -type d -empty -print -delete
	popd
	npm exec -c "electron-builder --linux --x64 --dir -c.electronDist=$electronDist \
	             -c.electronVersion=$electronVer"
}

package(){
	cd bitwarden/apps/desktop
	install -vDm644 dist/linux-unpacked/resources/app.asar -t "${pkgdir}/usr/lib/${pkgname}"
	install -vDm644 build/package.json -t "${pkgdir}/usr/lib/${pkgname}"
	install -vDm755 build/desktop_proxy -t "${pkgdir}/usr/lib/${pkgname}"
	cp -vr dist/linux-unpacked/resources/app.asar.unpacked -t "${pkgdir}/usr/lib/${pkgname}"

	for i in 16 32 64 128 256 512 1024; do
		install -vDm644 resources/icons/${i}x${i}.png "${pkgdir}/usr/share/icons/hicolor/${i}x${i}/apps/${pkgname}.png"
	done
	install -vDm644 resources/icon.png "${pkgdir}/usr/share/icons/hicolor/1024x1024/apps/${pkgname}.png"

	install -vDm755 "${srcdir}/${pkgname}.sh" "${pkgdir}/usr/bin/bitwarden-desktop"
	install -vDm644 "${srcdir}"/${pkgname}.desktop -t "${pkgdir}"/usr/share/applications
}
