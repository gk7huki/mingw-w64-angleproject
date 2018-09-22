# Maintainer: Huki <gk7huki@gmail.com>
# Contributor: Martchus <martchus@gmx.net>
# Contributor: Alexey Pavlov <alexpux@gmail.com>
# Contributor: ant32 <antreimer@gmail.com>
# Contributor: Filip Brcic <brcha@gna.org>
# Contributor: xantares
# Contributor: jellysheep <max DOT mail AT dameweb DOT de>
# Contributor: Ray Donnelly <mingw.android@gmail.com>

_angle_branch="#branch=chromium/3427"

pkgname='mingw-w64-angleproject'
pkgdesc='ANGLE project (mingw-w64)'
pkgver=2.1.r8420
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
        0000-build-fix.patch
        0001-include-import-library-and-use-def-file.patch
        0002-provide-windows-xp-support.patch
        0003-static-build-workaround.patch
        0004-redist.patch)
sha256sums=('SKIP'
            'c59de5a4705732f737cd5c5fa1c7ba372b695008803f7ff3d6428947badb7dfa'
            'acdf4672d473d76ac3d4b7ae7c4022feffbe4239467f80ebe68dd5c1a268d353'
            'e0a58c66ec14cdc1f9ba54f46fa57e8b246a291ff72185bcb4ec8c534e55a77d'
            '42a9e7d4e4365ce119fe5b036a088f226e35b0f6bb51913f3635e01fb53ef1a6'
            'dc9ac8ffa9df3bab614cb14d2743804e26a84d5cb1a644b66788919917b9c3cd')
_architectures=('i686-w64-mingw32' 'x86_64-w64-mingw32')

pkgver() {
  cd "${srcdir}"/angleproject
  local _major=$(head -n 14 src/common/version.h | grep 'ANGLE_MAJOR_VERSION' | sed -e 's/.* //' | tr '\n' '.' | sed 's/.$/\n/')
  local _minor=$(head -n 14 src/common/version.h | grep 'ANGLE_MINOR_VERSION' | sed -e 's/.* //' | tr '\n' '.' | sed 's/.$/\n/')
  printf "%s.%s.r%s" "$_major" "$_minor" "$(git rev-list --count HEAD)"
}

prepare() {
  cd "${srcdir}"/angleproject

  ### Fedora team patches ###
  patch -p1 -i ${srcdir}/0000-build-fix.patch

  # Make sure an import library is created and def file is used during the build
  patch -p1 -i ${srcdir}/0001-include-import-library-and-use-def-file.patch

  # Provide Windows XP support
  patch -p1 -i ${srcdir}/0002-provide-windows-xp-support.patch

  # Static build workaround
  patch -p1 -i ${srcdir}/0003-static-build-workaround.patch
  
  # Out of source directory fix
  patch -p1 -i ${srcdir}/0004-redist.patch
}

build() {
  cd "${srcdir}"/angleproject

  export PYTHON=/usr/bin/python2
  sed -i -e 's_python _python2 _g' -e 's_"python"_"python2"_g' -e "s_'python'_'python2'_g" -e 's_/usr/bin/python$_/usr/bin/python2_g' $(find -not \( -path "./.git*" -prune \) -type f)

  for _arch in ${_architectures[@]}; do
    mkdir -p "${srcdir}"/angleproject/build-${_arch}-shared
    #mkdir -p "${srcdir}"/angleproject/build-${_arch}-static

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

    #Build shared libraries
    gyp -D angle_enable_vulkan=0 -D use_ozone=0 -D OS=win -D MSVS_VERSION="" -D TARGET=${_target} --format make --generator-output="${srcdir}/angleproject/build-${_arch}-shared" --depth . -I "${srcdir}/angleproject/gyp/common.gypi" "${srcdir}/angleproject/src/angle.gyp"
    #gyp -D angle_enable_vulkan=0 -D use_ozone=0 -D OS=win -D MSVS_VERSION="" -D TARGET=${_target} --format make --generator-output="${srcdir}/angleproject/build-${_arch}-static" --depth . -I "${srcdir}/angleproject/gyp/common.gypi" "${srcdir}/angleproject/src/angle.gyp" -D angle_gl_library_type=static_library

    make -C "${srcdir}/angleproject/build-${_arch}-shared" -j$(($(nproc)+1)) V=1 BUILDTYPE=${_buildtype}
    #make -C "${srcdir}/angleproject/build-${_arch}-static" -j$(($(nproc)+1)) V=1 BUILDTYPE=${_buildtype}
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

    install "${srcdir}"/angleproject/build-${_arch}-shared/out/${_buildtype}/lib.target/libGLESv2.so "${pkgdir}/${_arch}"/bin/libGLESv2.dll
    install "${srcdir}"/angleproject/build-${_arch}-shared/out/${_buildtype}/lib.target/libGLESv1_CM.so "${pkgdir}/${_arch}"/bin/libGLESv1_CM.dll
    install "${srcdir}"/angleproject/build-${_arch}-shared/out/${_buildtype}/lib.target/libEGL.so "${pkgdir}/${_arch}"/bin/libEGL.dll
    ${_arch}-strip --strip-unneeded "${pkgdir}/${_arch}"/bin/*.dll

    install "${srcdir}"/angleproject/build-${_arch}-shared/libGLESv2.dll.a "${pkgdir}/${_arch}"/lib/
    install "${srcdir}"/angleproject/build-${_arch}-shared/libGLESv1_CM.dll.a "${pkgdir}/${_arch}"/lib/
    install "${srcdir}"/angleproject/build-${_arch}-shared/libEGL.dll.a "${pkgdir}/${_arch}"/lib/
    ${_arch}-strip --strip-unneeded "${pkgdir}/${_arch}"/lib/*.dll.a
    
    #install "${srcdir}"/angleproject/build-${_arch}-static/out/${_buildtype}/obj.target/src/{libGLESv2.a,libGLESv1_CM.a,libEGL.a} "${pkgdir}/${_arch}"/lib/
    #${_arch}-strip --strip-unneeded "${pkgdir}/${_arch}"/lib/{libGLESv2.a,libGLESv1_CM.a,libEGL.a}

    cp -Rv include/{EGL,GLES,GLES2,GLES3,KHR} "${pkgdir}/${_arch}"/include/
  done
}
