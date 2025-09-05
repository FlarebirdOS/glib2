pkgname=(
    glib2
    glib2-devel
    glib2-docs
)
pkgbase=glib2
pkgver=2.84.4
pkgrel=2
pkgdesc="Low level core library"
arch=('x86_64')
url="https://gitlab.gnome.org/GNOME/glib"
license=('LGPL-2.1-or-later')
depends=(
    'bash'
    'glibc'
    'libffi'
    'pcre2'
    'util-linux-libs'
    'zlib'
)
makedepends=(
    'bash-completion'
    'dbus'
    'gettext'
    'gobject-introspection'
    'libelf'
    'meson'
    'python'
    'python-docutils'
    'python-gi-docgen'
    'python-packaging'
    'python-setuptools'
    'util-linux'
)
source=(https://download.gnome.org/sources/glib/${pkgver%.*}/${pkgbase%2}-${pkgver}.tar.xz
    https://www.linuxfromscratch.org/patches/downloads/glib/glib-skip_warnings-1.patch
    gio-querymodules.hook
    glib-compile-schemas.hook)
sha256sums=(8a9ea10943c36fc117e253f80c91e477b673525ae45762942858aef57631bb90
    8f9ee9f4a6a08c49c9c912241c63d55b969950c49f4d40337c6fd9557b9daa1b
    b6fb5f07643c234bd0bde6c4899001effd270c17132e546cec535cb15771d269
    fe31399eb057d24a37062bcae6f88ca0778a91b85737f8110a03baa8bfc64fec)

prepare() {
    cd ${pkgbase%2}-${pkgver}

    patch -Np1 -i ${srcdir}/glib-skip_warnings-1.patch
}

build() {
    cd ${pkgbase%2}-${pkgver}

    local meson_args=(
        -D introspection=enabled
        -D glib_debug=disabled
        -D man-pages=enabled
        -D sysprof=disabled
        -D documentation=true
        -D selinux=disabled
    )

    # Produce more debug info: GLib has a lot of useful macros
    CFLAGS+=" -g3"
    CXXFLAGS+=" -g3"

    # use fat LTO objects for static libraries
    CFLAGS+=" -ffat-lto-objects"
    CXXFLAGS+=" -ffat-lto-objects"

    ${meson_options} "${meson_args[@]}"

    ${meson_build}
}

package_glib2() {
    cd ${pkgbase%2}-${pkgver}

    ${meson_install} ${pkgdir}

    install -Dt ${pkgdir}/usr/share/libalpm/hooks -m644 ${srcdir}/*.hook

    touch ${pkgdir}/usr/lib64/gio/modules/.keep

    python3 -m compileall -d /usr/share/glib-2.0/codegen \
        ${pkgdir}/usr/share/glib-2.0/codegen
    python3 -O -m compileall -d /usr/share/glib-2.0/codegen \
        ${pkgdir}/usr/share/glib-2.0/codegen

     # Split docs
    _pick docs ${pkgdir}/usr/share/doc

    # Split devel
    _pick devel ${pkgdir}/usr/bin/gdbus-codegen
    _pick devel ${pkgdir}/usr/bin/glib-{mkenums,genmarshal}
    _pick devel ${pkgdir}/usr/bin/gresource
    _pick devel ${pkgdir}/usr/bin/gtester{,-report}

    _pick devel ${pkgdir}/usr/share/gdb/
    _pick devel ${pkgdir}/usr/share/glib-2.0/gdb/
    _pick devel ${pkgdir}/usr/share/glib-2.0/codegen/

    _pick devel ${pkgdir}/usr/share/bash-completion/completions/gresource

    _pick devel ${pkgdir}/usr/share/man/man1/gdbus-codegen.1
    _pick devel ${pkgdir}/usr/share/man/man1/glib-{mkenums,genmarshal}.1
    _pick devel ${pkgdir}/usr/share/man/man1/gresource.1
    _pick devel ${pkgdir}/usr/share/man/man1/gtester{,-report}.1
}

package_glib2-devel() {
    pkgdesc+=" - development files"
    depends=(
        'glib2'
        'glibc'
        'libelf'
        'python'
        'python-packaging'
    )

    mv devel/* ${pkgdir}
}

package_glib2-docs() {
    pkgdesc+=" - documentation"
    depends=()
    license+=('LicenseRef-Public-Domain')

    mv docs/* ${pkgdir}
}
