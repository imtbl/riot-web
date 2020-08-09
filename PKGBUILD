# Maintainer: imtbl <imtbl at mser dot at>
pkgver=1.7.3.fork
pkgrel=1
pkgname=element-desktop-fork-git
_pkgname=element-web-fork-git
pkgdesc="A glossy Matrix collaboration client for the desktop with added custom emote support."
arch=('x86_64')
url="https://element.io"
license=('Apache')
depends=('electron')
makedepends=('git' 'nodejs' 'jq' 'yarn' 'npm' 'python' 'rust' 'sqlcipher' 'electron' 'moreutils')
conflicts=('element-desktop' 'element-desktop-git' 'element-web' 'riot-desktop-fork-git')
provides=('element-desktop')
backup=("etc/riot/config.json")
source=('element-web-fork-git::git://github.com/imtbl/element-web.git'
        'element-desktop::git://github.com/vector-im/element-desktop.git#tag=v1.7.3'
        'element-desktop.desktop'
        'element-desktop.sh')
sha256sums=('SKIP'
            'SKIP'
            '81354e663e354bd66b3f2bb303314b790bdf6d61c3d8e2df7407eb500885647d'
            '5988c98e8093a9a8e6c77d7135d809c590572a76b7b99bd524819f013594d7af')

prepare() {
  export HOME=$(mktemp -d) # Workaround to avoid conflicts when using `yarn link`

  cd "$srcdir/${_pkgname}"
  sed -i 's@"https://packages.riot.im/desktop/update/"@null@g' element.io/app/config.json

  cd "$srcdir/element-desktop"
  sed -i 's/"target": "deb"/"target": "dir"/g' package.json
  sed -i 's@"https://packages.riot.im/desktop/update/"@null@g' element.io/release/config.json
  jq '. + {emoteServerUrl: ""}' element.io/release/config.json | sponge element.io/release/config.json
  jq '. + {emoteServerAccessKey: ""}' element.io/release/config.json | sponge element.io/release/config.json
  jq '. + {defaultEmoteSize: "2em"}' element.io/release/config.json | sponge element.io/release/config.json
  jq '. + {largeEmoteSize: "4em"}' element.io/release/config.json | sponge element.io/release/config.json
}

build() {
  cd "$srcdir/${_pkgname}"
  yarn install --cache-folder "${srcdir}/yarn-cache"
  yarn build

  cd "$srcdir/element-desktop"
  yarn install --cache-folder "${srcdir}/yarn-cache"
  yarn build:native
  yarn build
}

package() {
  cd "$srcdir/${_pkgname}"

  install -d "${pkgdir}"/{usr/share/webapps,etc/webapps}/element

  cp -r webapp/* "${pkgdir}"/usr/share/webapps/element/
  install -Dm644 config.sample.json -t "${pkgdir}"/etc/webapps/element/
  ln -s /etc/webapps/element/config.json "${pkgdir}"/usr/share/webapps/element/
  echo "${pkgver}" > "${pkgdir}"/usr/share/webapps/element/version

  cd "$srcdir/element-desktop"

  install -d "${pkgdir}"{/usr/lib/element,/etc/webapps/element}

  cp -r dist/linux-unpacked/resources/* "${pkgdir}"/usr/lib/element/
  ln -s /usr/share/webapps/element "${pkgdir}"/usr/lib/element/webapp

  ln -s /etc/element/config.json "${pkgdir}"/etc/webapps/element/config.json
  install -Dm644 element.io/release/config.json -t "${pkgdir}"/etc/element/

  install -Dm644 "${srcdir}"/element-desktop.desktop "${pkgdir}"/usr/share/applications/element-desktop.desktop
  install -Dm755 "${srcdir}"/element-desktop.sh "${pkgdir}"/usr/bin/element-desktop

  install -Dm644 "$srcdir/${_pkgname}"/res/themes/element/img/logos/element-logo.svg "${pkgdir}"/usr/share/icons/hicolor/scalable/apps/element.svg

  for i in 16 24 48 64 96 128 256 512; do
    install -Dm644 build/icons/${i}x${i}.png "${pkgdir}"/usr/share/icons/hicolor/${i}x${i}/apps/element.png
  done
}
