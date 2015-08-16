# Maintainer: Filip Brcic <brcha@gna.org>
# OldMaintainer: rubenvb vanboxem <dottie> ruben <attie> gmail <dottie> com

pkgname=mingw-w64-gcc-qt5
pkgver=4.8.1
pkgrel=1
pkgdesc="Cross GCC for the MinGW-w64 cross-compiler"
arch=('i686' 'x86_64')
url="http://gcc.gnu.org"
license=('GPL' 'LGPL' 'FDL' 'custom')
groups=('mingw-w64-toolchain' 'mingw-w64')
depends=('zlib' 'libmpc' 'ppl' 'cloog' 'mingw-w64-crt-svn' 'mingw-w64-binutils' 'mingw-w64-winpthreads' 'mingw-w64-headers-svn' 'mingw-w64-headers-bootstrap')
makedepends=("gcc-ada=${pkgver}")
#checkdepends=('dejagnu') # Windows executables could run on Arch through bin_mft and Wine
optdepends=()
provides=('mingw-w64-gcc-base' 'mingw-w64-gcc')
conflicts=('mingw-w64-gcc-base' 'mingw-w64-gcc')
replaces=()
backup=()
options=('!strip' '!libtool' '!emptydirs' '!buildflags')
source=("ftp://gcc.gnu.org/pub/gcc/releases/gcc-${pkgver}/gcc-${pkgver}.tar.bz2"
        'gcc-make-xmmintrin-header-cplusplus-compatible.patch'
        'gcc-bug-56742-seh-uncaught-throw.patch')
md5sums=('3b2386c114cd74185aa3754b58a79304'
         'da6c9ba6baebe1286f3219d4181cdbb8'
         '3d5c4929cbe69911c308d5ec2d66da6d')

_targets="i686-w64-mingw32 x86_64-w64-mingw32"

prepare() {
  cd ${srcdir}/gcc-${pkgver}

  #do not install libiberty
  sed -i 's/install_to_$(INSTALL_DEST) //' libiberty/Makefile.in
  # hack! - some configure tests for header files using "$CPP $CPPFLAGS"
  sed -i "/ac_cpp=/s/\$CPPFLAGS/\$CPPFLAGS -O2/" {libiberty,gcc}/configure

  # The file xmmintrin.h doesn't contain an extern "C" part
  # This conflicts with mingw-w64 intrin.h and results in build
  # failure like this one in mingw-w64-qt5-qtbase:
  # /usr/lib/gcc/i686-w64-mingw32/4.8.0/include/xmmintrin.h:997:1: error: previous declaration of 'int _m_pextrw(__m64, int)' with 'C++' linkage
  # /usr/i686-w64-mingw32/include/intrin.h:561:28: error: conflicts with new declaration with 'C' linkage
  patch -p0 -i ${srcdir}/gcc-make-xmmintrin-header-cplusplus-compatible.patch

  # Optimization bug which can lead to uncaught throw (SEH related)
  patch -p1 -i ${srcdir}/gcc-bug-56742-seh-uncaught-throw.patch
}

build() {
  for _target in ${_targets}; do
    mkdir -p ${srcdir}/gcc-build-${_target} && cd ${srcdir}/gcc-build-${_target}
    
    ${srcdir}/gcc-${pkgver}/configure --prefix=/usr \
        --target=${_target} \
        --enable-languages=c,lto,c++,objc,obj-c++,fortran,ada \
        --enable-shared --enable-static \
        --enable-threads=win32 --enable-fully-dynamic-string \
        --with-system-zlib --with-ppl --enable-cloog-backend=isl \
        --enable-lto --disable-dw2-exceptions --enable-libgomp \
        --disable-nls \
        --disable-multilib --enable-checking=release
    make all
  done
}

package() {
  for _target in ${_targets}; do
    cd ${srcdir}/gcc-build-${_target}
    make DESTDIR=${pkgdir} install
    
    # Move DLLs to bindir
    mkdir -p ${pkgdir}/usr/${_target}/bin
    mv ${pkgdir}/usr/${_target}/lib/*.dll ${pkgdir}/usr/${_target}/bin/

    # Strip binaries
    ${_target}-strip --strip-unneeded ${pkgdir}/usr/${_target}/bin/*.dll
    mv ${pkgdir}/usr/${_target}/lib/*.dll.a ${pkgdir}/
    ${_target}-strip --strip-unneeded ${pkgdir}/*.dll.a
    ${_target}-strip --strip-debug ${pkgdir}/usr/${_target}/lib/*.a
    mv ${pkgdir}/*.dll.a ${pkgdir}/usr/${_target}/lib/
    strip --strip-all ${pkgdir}/usr/libexec/gcc/${_target}/${pkgver}/{cc1*,collect2,gnat1,f951,lto*}
  done
  strip --strip-all ${pkgdir}/usr/bin/*
  # remove unnecessary files
  rm -r ${pkgdir}/usr/share
}
