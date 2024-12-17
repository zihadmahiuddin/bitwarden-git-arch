# Maintainer: Alexander Epaneshnikov <alex19ep@archlinux.org>
# Contributor: libertylocked <libertylocked@disroot.org>

pkgname=bitwarden
pkgver=2024.12.1
pkgrel=1
_electronversion=32
pkgdesc='A secure and free password manager for all of your devices'
arch=('x86_64')
url='https://github.com/bitwarden/clients/tree/master/apps/desktop'
license=('GPL-3.0-only')
depends=("electron$_electronversion" 'libnotify' 'libsecret' 'org.freedesktop.secrets' 'libxtst' 'libxss' 'libnss_nis' 'argon2')
makedepends=('git' 'npm' 'python' 'python-setuptools' 'node-gyp' 'nodejs-lts-iron' 'jq' 'rust' 'fdupes')
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
sha512sums=('6a9546930e3a738cb282be8dea7bcc0632dbc5d67fa3c01e85d2682b8fe43a80a40ca49b0bd04c245bfc7961e5fcaa7b8155a7557203e5c3746215e99004d26b'
            '759db11cae26b8228000c98eb7bd21d0a46c964a858d27655f8f09114f5f7cba856623c3cad07424ba360e74144d9c0c050ee7219f8fe530cd9059d9f937f023'
            '0052ff95c0736eaa1f1b0790a65a9a935010b776e0b0d52ecf3ceb2c774277178872da1b6250a9189217fec4b4a64b9a4d30b129cc86c8a963228bba8723e27a'
            'f12482139463c6f471b49f789cd7f7e1748ecd06e169e5519edb29d762f456b9050871043336f98595b1cf15d99206bcad41f92a7145beb67c154e2a01e4c740'
            '56089fc612978e5f08c9745fb8a495eca719e744f99a711e7e1d0c5a3874f946da3d7f4174b340160bf8160a35bc2aa903fa64a3ba5bcc76088fdd407a97a10d'
            '6970261c24d8056b001f4ff308af7b9be67ef638be28f54f7c08ee4bed9ad895145a6cedf9a3cbda61b023104c50a6758a8bfcc8bc40097a298b98b60b1ffee7'
            '38ed719345ec0d5156a8fd9f2e4fadd836ae2179d5737c4b434b5062e8a7333d1c1fe4a89dada673ae4ae5d70cc1879bd8d84831edc178f08ab72117be432848'
            '5a4f6e09679399a6d11118ca1d6b9fc3b7e26594d285a264ea730062356c89ab5904e59c8465f90e6d901c134c8a91eb3cf123cae801d41a4b50ea0a12e832ce'
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
	npm ci
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
