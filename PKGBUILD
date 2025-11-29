# Maintainer: Toria <ninetailedtori@uwu.gal>
# Maintainer: Nick Østergaard <oe.nick at gmail dot com>

pkgname=kicad-git
pkgver=9.99.0.r3989.gc6d70fa3bb
pkgrel=1
pkgdesc="Electronic schematic and printed circuit board (PCB) design tools"
arch=('i686' 'x86_64')
url="https://kicad.org/"
license=('GPL-3.0-or-later')
depends=('boost-libs' 'curl' 'desktop-file-utils' 'glew' 'glm' 'libgit2'
         'libspnav' 'ngspice>=27' 'nng' 'opencascade' 'poppler' 'poppler-glib'
         'protobuf' 'python' 'python-wxpython' 'swig' 'unixodbc' 'wxwidgets-gtk3'
         'zstd')
makedepends=('boost' 'cmake' 'git' 'mesa' 'ninja' 'zlib')
optdepends=('kicad-library: for footprints')
conflicts=('kicad' 'kicad-bzr')
provides=('kicad')
install=kicad.install
source=("${pkgname}"'::git+https://gitlab.com/kicad/code/kicad.git')
b2sums=('SKIP')

pkgver() {
  cd "${srcdir}/${pkgname}"
  git describe --long | sed 's/\([^-]*-g\)/r\1/;s/-/./g'
}

build() {
  cd "${srcdir}/${pkgname}"
  mkdir -p build
  cd build
  cmake -GNinja .. -DCMAKE_BUILD_TYPE=RelWithDebInfo \
    -DCMAKE_INSTALL_PREFIX=/usr \
    -DCMAKE_INSTALL_LIBDIR=lib \
    -DKICAD_USE_EGL=ON \
    -DKICAD_USE_PCH=OFF \
    -DKICAD_USE_CMAKE_FINDPROTOBUF=0

  cmake --build .
}

package() {
  cd "${srcdir}/${pkgname}"
  DESTDIR="${pkgdir}" cmake --install build
}
