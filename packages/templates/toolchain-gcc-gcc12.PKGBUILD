# SPDX-License-Identifier: CC0-1.0
#
# SPDX-FileContributor: Adrian "asie" Siekierka, 2023

if [ "x$GCC_IS_LIBSTDCXX" = "xyes" ]; then
	pkgname=(toolchain-gcc-$GCC_TARGET-libstdcxx-picolibc)
	depends=(toolchain-gcc-$GCC_TARGET-gcc toolchain-gcc-$GCC_TARGET-picolibc-generic)
	arch=(any)
else
	pkgname=(toolchain-gcc-$GCC_TARGET-gcc toolchain-gcc-$GCC_TARGET-gcc-libs)
	depends=(runtime-gcc-libs runtime-musl toolchain-gcc-$GCC_TARGET-binutils)
	arch=("i686" "x86_64" "armv6h" "aarch64")
fi
pkgver=12.2.0
_gmpver=6.1.2
_mpfrver=4.0.2
_mpcver=1.1.0
_islver=0.21
epoch=
pkgdesc="The GNU Compiler Collection"
makedepends=(runtime-musl-dev)
groups=(toolchain-gcc-$GCC_TARGET)
url="https://gcc.gnu.org"
license=("GPL-3.0-or-later")
source=("http://ftp.gnu.org/gnu/gcc/gcc-$pkgver/gcc-$pkgver.tar.xz"
        "http://gmplib.org/download/gmp/gmp-$_gmpver.tar.xz"
        "http://www.mpfr.org/mpfr-$_mpfrver/mpfr-$_mpfrver.tar.xz"
        "http://ftp.gnu.org/gnu/mpc/mpc-$_mpcver.tar.gz"
	"https://libisl.sourceforge.io/isl-$_islver.tar.xz"
	"gcc12-poison-system-directories.patch")
sha256sums=(
	'e549cf9cf3594a00e27b6589d4322d70e0720cdd213f39beb4181e06926230ff'
	'87b565e89a9a684fe4ebeeddb8399dce2599f9c9049854ca8c0dfbdea0e21912'
	'1d3be708604eae0e42d578ba93b390c2a145f17743a744d8f3f8c2ad5855a38a'
	'6985c538143c1208dcb1ac42cedad6ff52e267b47e5f970183a3e75125b43c2e'
	'777058852a3db9500954361e294881214f6ecd4b594c00da5eee974cd6a54960'
	'ff867cbed6b359235122e5ee91708b931f8154dadc6cc3f6412be53eff76ca7b'
)

. "/wf/config/runtime-env-vars.sh"

prepare() {
	mkdir -p "gcc-build"
	cd "gcc-$pkgver"

	# Not necessary for GCC 12.2.0+ on musl
	# patch -p1 <../gcc12-poisoned-calloc-fix-libcc1.patch
	# patch -p1 <../gcc12-poisoned-calloc-fix-libgccjit.patch

	# Not strictly necessary, but a nice-to-have.
	patch -p1 <../gcc12-poison-system-directories.patch

	# HACK: hijack RTEMS's libstdc++ crossconfig for our own purposes (which has the dynamic feature checks we want)
	sed -i "s/\*-rtems\*/*-unknown*/" libstdc++-v3/configure

	ln -s ../"gmp-$_gmpver" gmp
	ln -s ../"mpfr-$_mpfrver" mpfr
	ln -s ../"mpc-$_mpcver" mpc
	ln -s ../"isl-$_islver" isl
}

build() {
	if [ "x$GCC_IS_LIBSTDCXX" = "xyes" ]; then
		build_libstdcxx_arg="--enable-libstdcxx"
		configure_cmd=../"gcc-$pkgver"/libstdc++-v3/configure
		export PATH=/opt/wonderful/toolchain/gcc-$GCC_TARGET/bin:$PATH
	else
		build_libstdcxx_arg="--disable-libstdcxx"
		configure_cmd=../"gcc-$pkgver"/configure
	fi
	cd gcc-build

	$configure_cmd \
		--prefix="/opt/wonderful/toolchain/gcc-$GCC_TARGET" \
		--target=$GCC_TARGET \
		--with-pkgversion="Wonderful toolchain" \
		--with-bugurl="http://github.com/WonderfulToolchain/wonderful-packages/issues" \
		--with-stage1-ldflags="$WF_RUNTIME_LDFLAGS" \
		--without-headers \
		--enable-plugins \
		--enable-poison-system-directories \
		--disable-bootstrap \
		--disable-gcov \
		--disable-nls \
		--disable-shared \
		--disable-werror \
		--disable-libquadmath \
		--disable-libssp \
		--disable-libstdcxx-pch \
		--disable-libstdcxx-threads \
		--disable-libstdcxx-verbose \
		--disable-libunwind-exceptions \
		--disable-threads \
		--enable-tls \
		--with-float=soft \
		--with-isl \
		$build_libstdcxx_arg \
		"${GCC_EXTRA_ARGS[@]}"

	make
}

package_toolchain-gcc-template-gcc() {
	cd gcc-build
	make DESTDIR="$pkgdir" install-gcc install-libcc1
	cd "$pkgdir"
	wf_relocate_path_to_destdir

	rm toolchain/gcc-$GCC_TARGET/share/info/dir
	rm toolchain/gcc-$GCC_TARGET/lib/gcc/$GCC_TARGET/$pkgver/include-fixed/README

	# HACK: As we don't build with a C library present, limits.h
	# assumes no such library is present.

	cd "$srcdir"/gcc-"$pkgver"/gcc
	cat limitx.h glimits.h limity.h > "$pkgdir"/toolchain/gcc-$GCC_TARGET/lib/gcc/$GCC_TARGET/$pkgver/include-fixed/limits.h

}

package_toolchain-gcc-template-gcc-libs() {
	pkgdesc="GCC-provided libraries"
	depends=("toolchain-gcc-$GCC_TARGET-binutils" "toolchain-gcc-$GCC_TARGET-gcc")
	options=(!strip)

	cd gcc-build
	make DESTDIR="$pkgdir" install-target-libgcc
	cd "$pkgdir"
	wf_relocate_path_to_destdir

	# HACK: Avoid conflict with -gcc package.
	rm toolchain/gcc-$GCC_TARGET/lib/gcc/$GCC_TARGET/*/include/unwind.h
}

package_toolchain-gcc-template-libstdcxx-picolibc() {
	pkgdesc="GCC-provided libstdc++, compiled for use with picolibc"
	options=(!strip)

	cd gcc-build
	make DESTDIR="$pkgdir" install
	cd "$pkgdir"
	wf_relocate_path_to_destdir
	rm -r toolchain/gcc-$GCC_TARGET/lib/*.py || true
	rm -r toolchain/gcc-$GCC_TARGET/share || true

	mkdir toolchain/gcc-$GCC_TARGET/$GCC_TARGET
	mv toolchain/gcc-$GCC_TARGET/include toolchain/gcc-$GCC_TARGET/$GCC_TARGET/
	mv toolchain/gcc-$GCC_TARGET/lib toolchain/gcc-$GCC_TARGET/$GCC_TARGET/
}
