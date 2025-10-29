pkgname=(
    glib2
    glib2-devel
    glib2-docs
)
pkgbase=glib2
pkgver=2.86.1
pkgrel=4
pkgdesc="Low level core library"
arch=('x86_64')
url="https://gitlab.gnome.org/GNOME/glib"
license=('LGPL-2.1-or-later')
depends=(
    'bash'
    'glibc'
    'libffi'
    'libsysprof-capture'
    'pcre2'
    'util-linux-libs'
    'zlib'
)
makedepends=(
    'dbus'
    'dconf'
    'gettext'
    'git'
    'gobject-introspection'
    'libelf'
    'meson'
    'python'
    'python-docutils'
    'python-gi-docgen'
    'python-packaging'
    'shared-mime-info'
    'util-linux'
)
source=(git+https://gitlab.gnome.org/GNOME/glib.git#tag=${pkgver}
    git+https://gitlab.gnome.org/GNOME/gvdb.git
    0001-glib-compile-schemas-Remove-noisy-deprecation-warnin.patch
    0002-gdesktopappinfo-Add-more-known-terminals.patch
    0003-meson.build-Avoid-linking-with-libatomic.patch
    gio-querymodules.hook
    glib-compile-schemas.hook)
sha256sums=(166dcac84931750a1dec84052e9bc1558ddb7b7bd7a019df6ffb3ae16c79b7c1
    SKIP
    380ff0a41e2efac4ee76e37bfb7bd17551b879dd29b653f59dbc9d38dcdc18b7
    324b923491c4030e4ca4ae2c3903004fbe026c558e1c0b2671cfadda06c82e87
    fd0a67057da4f6998c644970f7e7be64cdd837a3f1fdd62cd1c42d4fec242ec0
    b6fb5f07643c234bd0bde6c4899001effd270c17132e546cec535cb15771d269
    fe31399eb057d24a37062bcae6f88ca0778a91b85737f8110a03baa8bfc64fec)

prepare() {
    cd glib

    # Suppress noise from glib-compile-schemas.hook
    git apply -3 ${srcdir}/0001-glib-compile-schemas-Remove-noisy-deprecation-warnin.patch

    # Add ghostty and ptyxis to known terminals list
    # This is a downstream only patch; GNOME will not add new terminal emulators.
    # https://gitlab.gnome.org/GNOME/glib/-/issues/338#note_1076172
    git apply -3 ${srcdir}/0002-gdesktopappinfo-Add-more-known-terminals.patch

    # Drop dep on libatomic
    # https://gitlab.archlinux.org/archlinux/packaging/packages/qemu/-/issues/6
    # https://gitlab.gnome.org/GNOME/glib/-/issues/3407
    # https://gitlab.gnome.org/GNOME/glib/-/merge_requests/4774
    git apply -3 ${srcdir}/0003-meson.build-Avoid-linking-with-libatomic.patch

    git submodule init
    git submodule set-url subprojects/gvdb ${srcdir}/gvdb
    git -c protocol.file.allow=always -c protocol.allow=never submodule update
}

build() {
    cd glib

    local meson_args=(
        --default-library both
        -D documentation=true
        -D dtrace=disabled
        -D glib_debug=disabled
        -D introspection=enabled
        -D man-pages=enabled
        -D selinux=disabled
        -D sysprof=enabled
        -D systemtap=disabled
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
    install=glib2.install

    cd glib

    ${meson_install} ${pkgdir}

    install -Dt ${pkgdir}/usr/share/libalpm/hooks -m644 ${srcdir}/*.hook

    touch ${pkgdir}/usr/lib64/gio/modules/.keep

    python3 -m compileall -d /usr/share/glib-2.0/codegen ${pkgdir}/usr/share/glib-2.0/codegen
    python3 -O -m compileall -d /usr/share/glib-2.0/codegen ${pkgdir}/usr/share/glib-2.0/codegen

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
