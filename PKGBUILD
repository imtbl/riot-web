# Maintainer: Michael Serajnik <ms dot mserajnik dot at>
pkgver=v1.5.7.r198.gc0613fb4
pkgrel=1
pkgname=riot-desktop-fork-git
pkgdesc="A glossy Matrix collaboration client for the desktop."
arch=('any')
url="https://riot.im"
license=('Apache')
depends=('electron')
makedepends=('git' 'nodejs' 'jq' 'yarn' 'moreutils')
conflicts=('riot-desktop' 'riot-desktop-git' 'riot-web')
provides=('riot-desktop')
backup=("etc/riot/config.json")
source=('riot-desktop-fork-git::git://github.com/mserajnik/riot-web.git'
        'riot-desktop.desktop'
        'riot-desktop.sh')
md5sums=('SKIP'
         'SKIP'
         'SKIP')

pkgver() {
  cd "$srcdir/${pkgname}"

  ( set -o pipefail
    git describe --long 2>/dev/null | sed 's/\([^-]*-g\)/r\1/;s/-/./g' ||
    printf "r%s.%s" "$(git rev-list --count HEAD)" "$(git rev-parse --short HEAD)"
  )
}

prepare() {
  export HOME=$(mktemp -d) # Workaround to avoid conflicts when using `yarn link`
  cd "$srcdir/${pkgname}"
  sed -i 's@https://riot.im/download/desktop/update/@null@g' electron_app/riot.im/config.json
}

build() {
  cd "$srcdir/${pkgname}"

  yarn install --cache-folder "${srcdir}/npm-cache"

  jq ".compilerOptions.esModuleInterop = true" node_modules/matrix-react-sdk/tsconfig.json | sponge node_modules/matrix-react-sdk/tsconfig.json

  cd "$srcdir/${pkgname}/node_modules/matrix-react-sdk"
  yarn install --cache-folder "${srcdir}/npm-cache"
  yarn build

  cd "$srcdir/${pkgname}"
  yarn build
}

package() {
  cd "$srcdir/${pkgname}"

  install -d "${pkgdir}"/{usr/share/webapps,etc/webapps}/riot

  cp -r webapp/* "${pkgdir}"/usr/share/webapps/riot/
  install -Dm644 config.sample.json -t "${pkgdir}"/etc/webapps/riot/
  ln -s /etc/webapps/riot/config.json "${pkgdir}"/usr/share/webapps/riot/
  echo "${pkgver}" > "${pkgdir}"/usr/share/webapps/riot/version

  cd electron_app
  yarn --cache-folder "${srcdir}/npm-cache"
  cd ..

  install -d "${pkgdir}"{/usr/lib/riot/electron_app,/etc/webapps/riot}

  ln -s /usr/share/webapps/riot "${pkgdir}"/usr/lib/riot/webapp
  ln -s /etc/riot/config.json "${pkgdir}"/etc/webapps/riot/config.json

  install -Dm644 package.json -t "${pkgdir}"/usr/lib/riot
  cp -r electron_app/src "${pkgdir}"/usr/lib/riot/electron_app/
  cp -r electron_app/node_modules "${pkgdir}"/usr/lib/riot/electron_app/
  install -Dm644 electron_app/img/riot.png -t "${pkgdir}"/usr/lib/riot/electron_app/img
  install -Dm644 config.sample.json -t "${pkgdir}"/etc/riot/
  mv "${pkgdir}"/etc/riot/config.sample.json "${pkgdir}"/etc/riot/config.json

  install -Dm644 "${srcdir}"/riot-desktop.desktop "${pkgdir}"/usr/share/applications/riot.desktop
  install -Dm755 "${srcdir}"/riot-desktop.sh "${pkgdir}"/usr/bin/riot-desktop

  install -Dm644 "${srcdir}"/riot-desktop-fork-git/webapp/themes/riot/img/logos/riot-im-logo.svg "${pkgdir}"/usr/share/icons/hicolor/scalable/apps/riot.svg
  for i in 16 24 48 64 96 128 256 512; do
    install -Dm644 electron_app/build/icons/${i}x${i}.png "${pkgdir}"/usr/share/icons/hicolor/${i}x${i}/apps/riot.png
  done
}
