# Maintainer: Jan de Groot <jgc@archlinux.org>
# Maintainer: bohoomil <bohoomil@zoho.com>

pkgname=fontconfig-infinality-ultimate-git
_basename=fontconfig
_gitname=fontconfig-ultimate
pkgdesc="A library for configuring and customizing font access (includes generic fontconfig package and ultimate settings by bohoomil)."
url=('http://www.fontconfig.org/release/')
license=('GPL' 'MIT')
pkgver=2.11.1
pkgrel=8
arch=('i686' 'x86_64')
changelog=CHANGELOG
groups=('infinality-bundle')
depends=('expat' 'freetype2-infinality-ultimate')
options=('!libtool')
provides=('fontconfig=$pkgver' 'fontconfig-infinality' 'fontconfig-infinality-ultimate')
conflicts=('fontconfig' 'fontconfig-infinality' 'fontconfig-infinality-git' 'fontconfig-infinality-ultimate')
backup=('etc/fonts/fonts.conf'
        'etc/fonts/conf.avail.infinality/combi/30-metric-aliases-combi.conf'
        'etc/fonts/conf.avail.infinality/combi/37-repl-global-combi.conf'
        'etc/fonts/conf.avail.infinality/combi/60-latin-combi.conf'
        'etc/fonts/conf.avail.infinality/combi/65-non-latin-combi.conf'
        'etc/fonts/conf.avail.infinality/combi/66-aliases-wine-combi.conf'
        'etc/fonts/conf.avail.infinality/35-repl-custom.conf'
        'etc/fonts/conf.avail.infinality/36-repl-missing-glyphs.conf')
install=fontconfig-ultimate.install
source=(http://www.fontconfig.org/release/${_basename}-${pkgver}.tar.bz2
        'fontconfig-ultimate::git+http://github.com/bohoomil/fontconfig-ultimate#branch=master')
        #git://github.com/bohoomil/fontconfig-ultimate.git#branch=master
sha1sums=('08565feea5a4e6375f9d8a7435dac04e52620ff2'
          'SKIP')

# a nice page to test font matching:
# http://zipcon.net/~swhite/docs/computers/browsers/fonttest.html

#version() {
#  cd $_gitname
#  git log -1 --format="%cd" --date=short | sed 's|-|.|g'
#}

prepare() {
  cat << EOF

  Required dependency, freetype2-infinality-ultimate,
  you can either download as a pre-compiled binary package,
  or built on your own. See

  https://wiki.archlinux.org/index.php/Infinality-bundle+fonts

  for more information.
EOF

  read -p "Press [Enter] key to continue..."

  patches=(00-fonts.conf.in.patch
           01-configure.patch
           02-configure.ac.patch
           03-Makefile.in.patch
           04-Makefile.conf.d.patch
           05-Makefile.am.in.patch)

  # copy fontconfig-ib patches & stuff
  cd "${_gitname}"

  cp -r conf.d.infinality "${srcdir}/${_basename}-${pkgver}"/conf.d.infinality
  cp -r fontconfig_patches/*.patch "${srcdir}/${_basename}-${pkgver}"

  # prepare src
  cd "${srcdir}"/"${_basename}-${pkgver}"

  # infinality & post release fixes
  for patch in "${patches[@]}"; do
    patch -Np1 -i ${patch}
  done

  # fix for [bug 7752]
  # http://lists.freedesktop.org/archives/fontconfig-bugs/2014-April/000835.html
  patch -Np1 -i fc-cache.c.patch

  # make sure there's no rpath trouble and sane .so versioning -
  # FC and Gentoo do this as well
  aclocal
  libtoolize -f
  automake -afi
}

build() {
  cd "${_basename}-${pkgver}"

  ./configure --prefix=/usr \
    --sysconfdir=/etc \
    --with-templatedir=/etc/fonts/conf.avail \
    --with-templateinfdir=/etc/fonts/conf.avail.infinality \
    --with-xmldir=/etc/fonts \
    --localstatedir=/var \
    --disable-static \
    --with-default-fonts=/usr/share/fonts \
    --with-add-fonts=/usr/share/fonts
  make
}

#check() {
  #cd "${_basename}-${pkgver}"
  #make -k check
#}

package() {
  cd "${_basename}-${pkgver}"
  make DESTDIR="${pkgdir}" install

  #Install license
  install -m755 -d "${pkgdir}"/usr/share/licenses/"${_basename}"
  install -m644 COPYING "${pkgdir}"/usr/share/licenses/"${_basename}"

  #install infinality stuff
  cd "${srcdir}/${_gitname}"

  # copy presets
  cp -r fontconfig_patches/{combi,free,ms} \
    "${pkgdir}"/etc/fonts/conf.avail.infinality

  # install fc-presets
  install -m755 fontconfig_patches/"fc-presets" "${pkgdir}"/usr/bin/"fc-presets"

  # copy font settings
  install -m755 -d "${pkgdir}"/usr/share/doc/"${pkgname}"/fonts-settings
  cp fontconfig_patches/fonts-settings/*.conf  \
    "${pkgdir}"/usr/share/doc/"${pkgname}"/fonts-settings

  # copy documentation
  install -m755 -d "${pkgdir}"/usr/share/doc/"${pkgname}"
  cp -r doc "${pkgdir}"/usr/share/

  # install infinality-settings.sh
  install -m755 -d "${pkgdir}"/usr/share/doc/"${pkgname}"/freetype
  install -m755 freetype/infinality-settings.sh \
    "${pkgdir}"/usr/share/doc/"${pkgname}"/freetype/infinality-settings.sh

  find "${pkgdir}" -type d -name .git -exec rm -r '{}' +
}
