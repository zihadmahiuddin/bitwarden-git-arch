# Maintainer: Alexander Epaneshnikov <alex19ep@archlinux.org>
# Contributor: libertylocked <libertylocked@disroot.org>

pkgname=bitwarden
pkgver=2024.10.2
pkgrel=1
_electronversion=32
pkgdesc='A secure and free password manager for all of your devices'
arch=('x86_64')
url='https://github.com/bitwarden/clients/tree/master/apps/desktop'
license=('GPL-3.0-only')
depends=("electron$_electronversion" 'libnotify' 'libsecret' 'org.freedesktop.secrets' 'libxtst' 'libxss' 'libnss_nis' 'argon2')
makedepends=('git' 'npm' 'python' 'python-setuptools' 'node-gyp' 'nodejs-lts-iron' 'jq' 'rust')
source=(bitwarden::git+https://github.com/bitwarden/clients.git#tag=desktop-v$pkgver
        messaging.main.ts.patch
        nativelib.patch
        native-messaging.main.ts.patch
        remove-argon2-browser.patch
        system-libargon2.patch
        no-sourcemaps.patch
        remove-unnecessary-deps.patch
        ${pkgname}.sh
        ${pkgname}.desktop)
sha512sums=('d5d40a0058ca45cd3d923c6de50e1ec625c80eaa84a482683b68cbcd6cca043d2d674be804f57e22e1809a5ca0663dfaa646a36ec92a46c1cfde9e5f457009ca'
            '759db11cae26b8228000c98eb7bd21d0a46c964a858d27655f8f09114f5f7cba856623c3cad07424ba360e74144d9c0c050ee7219f8fe530cd9059d9f937f023'
            '0052ff95c0736eaa1f1b0790a65a9a935010b776e0b0d52ecf3ceb2c774277178872da1b6250a9189217fec4b4a64b9a4d30b129cc86c8a963228bba8723e27a'
            'f12482139463c6f471b49f789cd7f7e1748ecd06e169e5519edb29d762f456b9050871043336f98595b1cf15d99206bcad41f92a7145beb67c154e2a01e4c740'
            '56089fc612978e5f08c9745fb8a495eca719e744f99a711e7e1d0c5a3874f946da3d7f4174b340160bf8160a35bc2aa903fa64a3ba5bcc76088fdd407a97a10d'
            'df60305a0fec1ac296f2df21e195545ae7134bd50b17093447c2648670868d820d0b5e2b1def12814eb5c87f95a2dfcfd665749d2e4925b48ec61707e11e1b6f'
            '38ed719345ec0d5156a8fd9f2e4fadd836ae2179d5737c4b434b5062e8a7333d1c1fe4a89dada673ae4ae5d70cc1879bd8d84831edc178f08ab72117be432848'
            '89f2b4d07b2db4de837cf959667ce883bd775a9d7eaa013342fe942e21ce2248a24c1f5acd253ce40a69bcef46c1c0dabcfb069dc89f25b7aa1d8ae8c795d1d7'
            '09acb1f4a7fb04fda120eda79ee847f285a421bc5bf5c3d42c78767f01f2051984b928019707c7c59e48e87728ab45ee2a98ff7ccee6b4e14bfdff93cd1106f0'
            'fdc047aadc1cb947209d7ae999d8a7f5f10ae46bf71687080bc13bc3082cc5166bbbe88c8f506b78adb5d772f5366ec671b04a2f761e7d7f249ebe5726681e30')

prepare() {
	cd bitwarden/apps/desktop

	export npm_config_build_from_source=true
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
	npm install
	patch -p1 -i "$srcdir/system-libargon2.patch"
}

build() {
	cd bitwarden/apps/desktop
	electronDist=/usr/lib/electron$_electronversion
	electronVer=$(electron$_electronversion --version | tail -c +2)
	export npm_config_build_from_source=true
	export npm_config_cache="$srcdir/npm_cache"
	export ELECTRON_SKIP_BINARY_DOWNLOAD=1
	export CXXFLAGS="$CXXFLAGS -Wno-error"
	pushd desktop_native
	cargo rustc --release --package desktop_napi --lib --crate-type cdylib
RUSTFLAGS="$RUSTFLAGS -Crelocation-model=pie" cargo rustc --release --package desktop_proxy --bin desktop_proxy
	mv -v target/release/*.so napi/desktop_napi.node
	popd
	npm run build
	npm run clean:dist
	mv -v desktop_native/target/release/desktop_proxy build/desktop_proxy
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
