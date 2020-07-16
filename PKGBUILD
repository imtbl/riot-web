# Maintainer: imtbl <imtbl at mser dot at>
pkgver=1.7.1.fork
pkgrel=1
pkgname=riot-desktop-fork-git
_pkgname=riot-web-fork-git
pkgdesc="A glossy Matrix collaboration client for the desktop with added custom emote support."
arch=('any')
url="https://element.io"
license=('Apache')
depends=('electron' 'sqlcipher')
makedepends=('git' 'nodejs' 'jq' 'yarn' 'npm' 'python' 'rust' 'moreutils')
conflicts=('riot-desktop' 'riot-desktop-git' 'riot-web')
provides=('riot-desktop')
backup=("etc/riot/config.json")
source=('riot-web-fork-git::git://github.com/imtbl/riot-web.git'
        'riot-desktop::git://github.com/vector-im/riot-desktop.git#tag=v1.7.1'
        'riot-desktop.desktop'
        'riot-desktop.sh')
sha256sums=('SKIP'
            'SKIP'
            'b94e5d51831cf57729c96e716cdda3a034a1dda7a0f9e2e6fbd040be2c862604'
            'a6cc599b226357a6e219d17b29834aa34b993029c5c83e46402117f10fa4f91e')

prepare() {
  export HOME=$(mktemp -d) # Workaround to avoid conflicts when using `yarn link`

  cd "$srcdir/${_pkgname}"
  sed -i 's@"https://packages.riot.im/desktop/update/"@null@g' element.io/app/config.json

  cd "$srcdir/riot-desktop"
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

  cd "$srcdir/riot-desktop"
  yarn install --cache-folder "${srcdir}/yarn-cache"
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
  install -Dm644 element.io/release/config.json -t "${pkgdir}"/etc/riot/

  install -Dm644 "${srcdir}"/riot-desktop.desktop "${pkgdir}"/usr/share/applications/riot-desktop.desktop
  install -Dm755 "${srcdir}"/riot-desktop.sh "${pkgdir}"/usr/bin/riot-desktop

  install -Dm644 "$srcdir/${_pkgname}"/res/themes/element/img/logos/element-logo.svg "${pkgdir}"/usr/share/icons/hicolor/scalable/apps/riot.svg
  for i in 16 24 48 64 96 128 256 512; do
    install -Dm644 build/icons/${i}x${i}.png "${pkgdir}"/usr/share/icons/hicolor/${i}x${i}/apps/riot.png
  done
}
