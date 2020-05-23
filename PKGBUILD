# Maintainer: Jan de Groot <jgc@archlinux.org>
# Maintainer: Andreas Radke <andyrtr@archlinux.org>

pkgbase=mesa
pkgname=('libva-mesa-driver')
pkgdesc="An open-source implementation of the OpenGL specification"
pkgver=20.0.7
pkgrel=99
arch=('x86_64')
makedepends=('python-mako' 'libxml2' 'libx11' 'xorgproto' 'libdrm' 'libxshmfence' 'libxxf86vm'
             'libxdamage' 'libvdpau' 'libva' 'wayland' 'wayland-protocols' 'zstd'
             'elfutils' 'llvm' 'libomxil-bellagio' 'libclc' 'clang' 'libglvnd' 'libunwind' 'lm_sensors'
             'libxrandr' 'valgrind' 'glslang' 'meson')
url="https://www.mesa3d.org/"
license=('custom')
source=(https://mesa.freedesktop.org/archive/mesa-${pkgver}.tar.xz{,.sig}
        convert-interlaced-nv12-to-progressive.patch
        0001-swr-Fix-build-with-GCC-10.patch
        0001-omx-fix-build-with-gcc-10.patch
        0001-meson-Disable-GCC-s-dead-store-elimination-for-memor.patch
        LICENSE)
sha512sums=('00baae50f14bf2b08b5654dffb11cf67499dc1825e1700b137fb5719e767e0e78e789979df2c194f677ea9c5e531f34965d47b9e37c239944c38d0570c7a9685'
            'SKIP'
            '0b72088f8c41259ddcb83468345ee72e1a928833c284415afec82a96daf112697732d4cc659f22e39780fb329459adedfb9ff6b76839675e20136147a1c622ad'
            '296a7502e959ccd2a6f1279878c0562a853ecdd78b5960196fc8f99ed8dd995c6e1106551aef7a53db891295235ca55676788e7cf78e336e2d5ee49e4e463be5'
            'e1f0fa2a8802184580d9d95777f02a1c35bf71c3ab380d88e5b9268f84c2ac338fa517d20065094b7764490bbbfb290c1c5ad6dec6d27f3dbf737dfa0b6c7263'
            '566ad6f4195124b4af05d4bba1e01cc5e9ac466f11ddd900a2f5bfd830aa19cdccb3f9625901340b5fe62e7d8ea50aa336ab5031a658fe90916d847b2e9946e0'
            'f9f0d0ccf166fe6cb684478b6f1e1ab1f2850431c06aa041738563eb1808a004e52cdec823c103c9e180f03ffc083e95974d291353f0220fe52ae6d4897fecc7')
validpgpkeys=('8703B6700E7EE06D7A39B8D6EDAE37B02CEB490D'  # Emil Velikov <emil.l.velikov@gmail.com>
              '946D09B5E4C9845E63075FF1D961C596A7203456'  # Andres Gomez <tanty@igalia.com>
              'E3E8F480C52ADD73B278EE78E1ECBE07D7D70895'  # Juan Antonio Suárez Romero (Igalia, S.L.) <jasuarez@igalia.com>
              'A5CC9FEC93F2F837CB044912336909B6B25FADFA'  # Juan A. Suarez Romero <jasuarez@igalia.com>
              '71C4B75620BC75708B4BDB254C95FAAB3EB073EC'  # Dylan Baker <dylan@pnwbakers.com>
	      '57551DE15B968F6341C248F68D8E31AFC32428A6') # Eric Engestrom <eric@engestrom.ch>

prepare() {
  cd mesa-$pkgver
  patch --forward --strip=1 --input="${srcdir}/convert-interlaced-nv12-to-progressive.patch"
  # fix building with gcc 10
  patch -Np1 -i ../0001-swr-Fix-build-with-GCC-10.patch
  patch -Np1 -i ../0001-omx-fix-build-with-gcc-10.patch
  # LTO fix
  patch -Np1 -i ../0001-meson-Disable-GCC-s-dead-store-elimination-for-memor.patch
}

build() {
  arch-meson mesa-$pkgver build \
    -D b_lto=true \
    -D b_ndebug=true \
    -D platforms=x11,wayland,drm,surfaceless \
    -D dri-drivers=i915,i965,r100,r200,nouveau \
    -D gallium-drivers=r300,r600,radeonsi,nouveau,virgl,svga,swrast,swr,iris \
    -D vulkan-drivers=amd,intel \
    -D vulkan-overlay-layer=true \
    -D swr-arches=avx,avx2 \
    -D dri3=true \
    -D egl=true \
    -D gallium-extra-hud=true \
    -D gallium-nine=true \
    -D gallium-omx=bellagio \
    -D gallium-opencl=icd \
    -D gallium-va=true \
    -D gallium-vdpau=true \
    -D gallium-xa=true \
    -D gallium-xvmc=false \
    -D gbm=true \
    -D gles1=false \
    -D gles2=true \
    -D glvnd=true \
    -D glx=dri \
    -D libunwind=true \
    -D llvm=true \
    -D lmsensors=true \
    -D osmesa=gallium \
    -D shared-glapi=true \
    -D valgrind=true

  # Print config
  meson configure build

  ninja -C build

  # fake installation to be seperated into packages
  # outside of fakeroot but mesa doesn't need to chown/mod
  DESTDIR="${srcdir}/fakeinstall" ninja -C build install
}

_install() {
  local src f dir
  for src; do
    f="${src#fakeinstall/}"
    dir="${pkgdir}/${f%/*}"
    install -m755 -d "${dir}"
    mv -v "${src}" "${dir}/"
  done
}

package_libva-mesa-driver() {
  pkgdesc="VA-API implementation for gallium"
  depends=('libdrm' 'libx11' 'llvm-libs' 'expat' 'libelf' 'libxshmfence' 'zstd')

  _install fakeinstall/usr/lib/dri/*_drv_video.so

  install -m644 -Dt "${pkgdir}/usr/share/licenses/${pkgname}" LICENSE
}

