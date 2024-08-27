# Maintainer: Alexander Epaneshnikov <alex19ep@archlinux.org>
# Contributor: libertylocked <libertylocked@disroot.org>

pkgname=bitwarden
pkgver=2024.8.0
pkgrel=2
_electronversion=31
pkgdesc='A secure and free password manager for all of your devices'
arch=('x86_64')
url='https://github.com/bitwarden/clients/tree/master/apps/desktop'
license=('GPL3')
depends=("electron$_electronversion" 'libnotify' 'libsecret' 'org.freedesktop.secrets' 'libxtst' 'libxss' 'libnss_nis')
makedepends=('git' 'npm' 'python' 'python-setuptools' 'node-gyp' 'nodejs-lts-iron' 'jq' 'rust')
source=(bitwarden::git+https://github.com/bitwarden/clients.git#tag=desktop-v$pkgver
        messaging.main.ts.patch
        nativelib.patch
        native-messaging.main.ts.patch
        ${pkgname}.sh
        ${pkgname}.desktop)
sha512sums=('7cef95c6391f0b4d3231f8064336df1382bbcf2dee0b893efdf9c2c34bf7d17b63fca8534bfca3f49b452ffbaf3983bb69797f786222adbfd48f619fd86438db'
            'babcae0dba4d036e5d2cd04d8932b63253bc7b27b14d090932066e9d39383f7565c06d72dae9f96e741b494ef7e50a1fe7ec33905aa3124b427a8bf404df5762'
            'd2ca6c5fa428a74d463e21573ad0dae626eaec14d8318228320967d59760a86a641369880b79aa637fb1572c49b9c9f8c7cdfe9738be5a4bc9118249576311a2'
            '500396e1626115ed3fe47270f32500e733517912bde7c7fd7d28d5e063bcae9ba760944d442a97479f226fa0fbb596095bb08aa74874c0bc48ce9bd752419b50'
            '09acb1f4a7fb04fda120eda79ee847f285a421bc5bf5c3d42c78767f01f2051984b928019707c7c59e48e87728ab45ee2a98ff7ccee6b4e14bfdff93cd1106f0'
            'fdc047aadc1cb947209d7ae999d8a7f5f10ae46bf71687080bc13bc3082cc5166bbbe88c8f506b78adb5d772f5366ec671b04a2f761e7d7f249ebe5726681e30')

prepare() {
	cd bitwarden/apps/desktop

	export npm_config_build_from_source=true
	export npm_config_cache="$srcdir/npm_cache"
	export ELECTRON_SKIP_BINARY_DOWNLOAD=1

	# This patch is required to make "Start automatically on login" work
	patch --strip=1 src/main/messaging.main.ts "$srcdir/messaging.main.ts.patch"

	# This patch is required to make "browser integration" work
	patch --strip=1 src/main/native-messaging.main.ts "$srcdir/native-messaging.main.ts.patch"

	# Patch build to make it work with system electron
	export SYSTEM_ELECTRON_VERSION=$(electron$_electronversion -v | sed 's/v//g')
	export ELECTRONVERSION=$_electronversion
	sed -i "s|@electronversion@|${ELECTRONVERSION}|" "$srcdir/bitwarden.sh"
	cd ../../
	jq < package.json \
	   '.devDependencies["electron"]=$ENV.SYSTEM_ELECTRON_VERSION' \
	   > package.json.patched
	mv package.json.patched package.json
	patch --strip=1 apps/desktop/desktop_native/napi/index.js "$srcdir/nativelib.patch"
	npm install
}

build() {
	cd bitwarden/apps/desktop
	electronDist=/usr/lib/electron$_electronversion
	electronVer=$(electron$_electronversion --version | tail -c +2)
	export npm_config_build_from_source=true
	export npm_config_cache="$srcdir/npm_cache"
	export ELECTRON_SKIP_BINARY_DOWNLOAD=1
    export CXXFLAGS="$CXXFLAGS -Wno-error"
	pushd desktop_native/napi
	npm run build
	popd
	npm run build
	npm run clean:dist
	npm exec -c "electron-builder --linux --x64 --dir -c.electronDist=$electronDist \
	             -c.electronVersion=$electronVer"
}

package(){
	cd bitwarden/apps/desktop
	install -vDm644 dist/linux-unpacked/resources/app.asar -t "${pkgdir}/usr/lib/${pkgname}"
	install -vDm644 build/package.json -t "${pkgdir}/usr/lib/${pkgname}"
	cp -vr dist/linux-unpacked/resources/app.asar.unpacked -t "${pkgdir}/usr/lib/${pkgname}"

	for i in 16 32 64 128 256 512 1024; do
		install -vDm644 resources/icons/${i}x${i}.png "${pkgdir}/usr/share/icons/hicolor/${i}x${i}/apps/${pkgname}.png"
	done
	install -vDm644 resources/icon.png "${pkgdir}/usr/share/icons/hicolor/1024x1024/apps/${pkgname}.png"

	install -vDm755 "${srcdir}/${pkgname}.sh" "${pkgdir}/usr/bin/bitwarden-desktop"
	install -vDm644 "${srcdir}"/${pkgname}.desktop -t "${pkgdir}"/usr/share/applications
}
