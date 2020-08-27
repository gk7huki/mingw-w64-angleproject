# Maintainer: Huki <gk7huki@gmail.com>
# Contributor: Martchus <martchus@gmx.net>
# Contributor: Alexey Pavlov <alexpux@gmail.com>
# Contributor: ant32 <antreimer@gmail.com>
# Contributor: Filip Brcic <brcha@gna.org>
# Contributor: xantares
# Contributor: jellysheep <max DOT mail AT dameweb DOT de>
# Contributor: Ray Donnelly <mingw.android@gmail.com>

_angle_branch="#branch=chromium/3497"

pkgname='mingw-w64-angleproject'
pkgdesc='ANGLE project (mingw-w64)'
pkgver=2.1.r8796
pkgrel=1
arch=('any')
url='https://chromium.googlesource.com/angle/angle'
license=('BSD')
depends=('mingw-w64-crt')
makedepends=('mingw-w64-gcc' 'gyp' 'git' 'python' 'python2' 'python2-setuptools')
provides=("mingw-w64-angleproject")
conflicts=("mingw-w64-angleproject")
options=('strip' '!buildflags' 'staticlibs')
source=("angleproject"::git+https://chromium.googlesource.com/angle/angle$_angle_branch
        0001-build-fix.patch
        0002-windows-xp-support.patch
        0003-add-link-libraries.patch
        0004-use-import-library-and-def-file.patch
        0005-fix-python2-references.patch)
sha256sums=('SKIP'
            '3b08b59424474aa1d54167fe96643e4e1dd54cc0fed654edbc12478b7b316e25'
            '14c2c731f341a9c6274efb4958f1ec93e72ea9fb4c3e3c648551608230b18b78'
            'f9fb1f87019dd6a6f7d9225dbc8d29fa3ce656a1510a00cf48f92c16b4c7b427'
            '61b5bf19aa73c1de54caf4b13170a48e89d8337c852f6f54eda2c49adbfaed41'
            '0f777b38b4e0f9befaa5db4df0530605b0f812bba2ed0af03d776a76f2bb272e')
_architectures=('i686-w64-mingw32' 'x86_64-w64-mingw32')

pkgver() {
  cd "${srcdir}"/angleproject
  local _major=$(head -n 14 src/common/version.h | grep 'ANGLE_MAJOR_VERSION' | sed -e 's/.* //' | tr '\n' '.' | sed 's/.$/\n/')
  local _minor=$(head -n 14 src/common/version.h | grep 'ANGLE_MINOR_VERSION' | sed -e 's/.* //' | tr '\n' '.' | sed 's/.$/\n/')
  printf "%s.%s.r%s" "$_major" "$_minor" "$(git rev-list --count HEAD)"
}

prepare() {
  cd "${srcdir}"/angleproject

  # MinGW build fixes
  patch -p1 -i ${srcdir}/0001-build-fix.patch

  # Provide Windows XP support
  patch -p1 -i ${srcdir}/0002-windows-xp-support.patch

  # Add missing link libraries
  patch -p1 -i ${srcdir}/0003-add-link-libraries.patch

  # Make sure an import library is created and def file is used
  patch -p1 -i ${srcdir}/0004-use-import-library-and-def-file.patch
  
  # Change references to python into python2
  patch -p1 -i ${srcdir}/0005-fix-python2-references.patch
}

build() {
  cd "${srcdir}"/angleproject

  export PYTHON=/usr/bin/python2

  for _arch in ${_architectures[@]}; do
    mkdir -p "${srcdir}"/angleproject/build_${_arch}_shared

    # Set ar/compiler and architecture specific compiler flags
    export CC="${_arch}-gcc"
    export CXX="${_arch}-g++"
    export AR="${_arch}-ar"
    export AS="${_arch}-as"
    export RANLIB="${_arch}-ranlib"
    export LINK="${_arch}-g++"
    export CXXFLAGS="-O2 -pipe -Wall -std=c++14 -msse2 -DUNICODE -D_UNICODE"
    export LDFLAGS="-static"

    if [[ "${_arch}" == "i686-w64-mingw32" ]]; then
      _target="win32"
      _buildtype="Release"
    else
      _target="win64"
      _buildtype="Release_x64"
    fi

    # Build shared libraries
    gyp -D angle_enable_vulkan=0 -D use_ozone=0 -D OS=win -D MSVS_VERSION="" -D TARGET=${_target} --format make --generator-output="${srcdir}/angleproject/build_${_arch}_shared" --depth . -I "${srcdir}/angleproject/gyp/common.gypi" "${srcdir}/angleproject/src/angle.gyp"
    make -C "${srcdir}/angleproject/build_${_arch}_shared" libEGL -j$(($(nproc)+1)) V=1 BUILDTYPE=${_buildtype}
  done
}

package() {
  cd "${srcdir}"/angleproject

  for _arch in ${_architectures[@]}; do
    mkdir -p "${pkgdir}/${_arch}"/{bin,lib,include}

    if [[ "${_arch}" == "i686-w64-mingw32" ]]; then
      _buildtype="Release"
    else
      _buildtype="Release_x64"
    fi

    install "${srcdir}"/angleproject/build_${_arch}_shared/out/${_buildtype}/lib.target/libGLESv2.so "${pkgdir}/${_arch}"/bin/libGLESv2.dll
    install "${srcdir}"/angleproject/build_${_arch}_shared/out/${_buildtype}/lib.target/libEGL.so "${pkgdir}/${_arch}"/bin/libEGL.dll
    ${_arch}-strip --strip-unneeded "${pkgdir}/${_arch}"/bin/*.dll

    install "${srcdir}"/angleproject/build_${_arch}_shared/out/libGLESv2.dll.a "${pkgdir}/${_arch}"/lib/libGLESv2.dll.a
    install "${srcdir}"/angleproject/build_${_arch}_shared/out/libEGL.dll.a "${pkgdir}/${_arch}"/lib/libEGL.dll.a
    ${_arch}-strip --strip-unneeded "${pkgdir}/${_arch}"/lib/*.dll.a

    cp -Rv include/{EGL,GLES2,GLES3} "${pkgdir}/${_arch}"/include/
  done
}
