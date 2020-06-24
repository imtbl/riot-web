# Maintainer: Michael Serajnik <ms dot mserajnik dot at>
pkgver=v1.5.7.r943.gd47edea3
pkgrel=1
pkgname=riot-desktop-fork-git
_pkgname=riot-web-fork-git
pkgdesc="A glossy Matrix collaboration client for the desktop."
arch=('any')
url="https://riot.im"
license=('Apache')
depends=('electron' 'sqlcipher')
makedepends=('git' 'nodejs' 'jq' 'yarn' 'npm' 'python' 'rust' 'moreutils')
conflicts=('riot-desktop' 'riot-desktop-git' 'riot-web')
provides=('riot-desktop')
backup=("etc/riot/config.json")
source=('riot-web-fork-git::git://github.com/mserajnik/riot-web.git'
        'riot-desktop::git://github.com/vector-im/riot-desktop.git#tag=v1.6.4'
        'riot-desktop.desktop'
        'riot-desktop.sh')
md5sums=('SKIP'
         'SKIP'
         'SKIP'
         'SKIP')

pkgver() {
  cd "$srcdir/${_pkgname}"

  ( set -o pipefail
    git describe --long 2>/dev/null | sed 's/\([^-]*-g\)/r\1/;s/-/./g' ||
    printf "r%s.%s" "$(git rev-list --count HEAD)" "$(git rev-parse --short HEAD)"
  )
}

prepare() {
  export HOME=$(mktemp -d) # Workaround to avoid conflicts when using `yarn link`

  cd "$srcdir/${_pkgname}"
  sed -i 's@"https://packages.riot.im/desktop/update/"@null@g' riot.im/app/config.json

  cd "$srcdir/riot-desktop"
  sed -i 's/"target": "deb"/"target": "dir"/g' package.json
  sed -i 's@"https://packages.riot.im/desktop/update/"@null@g' riot.im/release/config.json
  jq '. + {emoteServerUrl: ""}' riot.im/release/config.json | sponge riot.im/release/config.json
  jq '. + {emoteServerAccessKey: ""}' riot.im/release/config.json | sponge riot.im/release/config.json
  jq '. + {defaultEmoteSize: "2em"}' riot.im/release/config.json | sponge riot.im/release/config.json
  jq '. + {largeEmoteSize: "4em"}' riot.im/release/config.json | sponge riot.im/release/config.json
}

build() {
  cd "$srcdir/${_pkgname}"

  yarn install --cache-folder "${srcdir}/npm-cache"

  jq ".compilerOptions.esModuleInterop = true" node_modules/matrix-react-sdk/tsconfig.json | sponge node_modules/matrix-react-sdk/tsconfig.json

  cd "$srcdir/${_pkgname}/node_modules/matrix-react-sdk"
  yarn install --cache-folder "${srcdir}/npm-cache"
  yarn build

  cd "$srcdir/${_pkgname}"
  yarn build

  cd "$srcdir/riot-desktop"
  yarn install --cache-folder "${srcdir}/npm-cache"
  yarn build:native
  yarn build
}

package() {
  cd "$srcdir/${_pkgname}"

  install -d "${pkgdir}"/{usr/share/webapps,etc/webapps}/riot

  cp -r webapp/* "${pkgdir}"/usr/share/webapps/riot/
  install -Dm644 config.sample.json -t "${pkgdir}"/etc/webapps/riot/
  ln -s /etc/webapps/riot/config.json "${pkgdir}"/usr/share/webapps/riot/
  echo "${pkgver}" > "${pkgdir}"/usr/share/webapps/riot/version

  cd "$srcdir/riot-desktop"

  install -d "${pkgdir}"{/usr/lib/riot,/etc/webapps/riot}

  cp -r dist/linux-unpacked/resources/* "${pkgdir}"/usr/lib/riot/
  ln -s /usr/share/webapps/riot "${pkgdir}"/usr/lib/riot/webapp

  ln -s /etc/riot/config.json "${pkgdir}"/etc/webapps/riot/config.json
  install -Dm644 riot.im/release/config.json -t "${pkgdir}"/etc/riot/

  install -Dm644 "${srcdir}"/riot-desktop.desktop "${pkgdir}"/usr/share/applications/riot-desktop.desktop
  install -Dm755 "${srcdir}"/riot-desktop.sh "${pkgdir}"/usr/bin/riot-desktop

  install -Dm644 "$srcdir/${_pkgname}"/res/themes/riot/img/logos/riot-im-logo.svg "${pkgdir}"/usr/share/icons/hicolor/scalable/apps/riot.svg
  for i in 16 24 48 64 96 128 256 512; do
    install -Dm644 build/icons/${i}x${i}.png "${pkgdir}"/usr/share/icons/hicolor/${i}x${i}/apps/riot.png
  done
}
