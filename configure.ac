AC_PREREQ([2.69])
AC_INIT([usbguard],
        m4_esyscmd_s([cat VERSION]),
        [usbguard@usbguard.org])
AC_CONFIG_SRCDIR([src/])
AC_CONFIG_HEADERS([src/build-config.h.in])
AC_CONFIG_MACRO_DIR([m4])
AC_CONFIG_AUX_DIR([config])

AM_INIT_AUTOMAKE([subdir-objects foreign tar-pax])

# If automake supports "silent rules", enable them by default
m4_ifdef([AM_SILENT_RULES], [AM_SILENT_RULES([yes])])

#
# Compiler flags
#
EXTERNAL_CXXFLAGS="$CXXFLAGS"
EXTERNAL_CFLAGS="$CFLAGS"
EXTERNAL_CPPFLAGS="$CPPFLAGS"

COMMON_WARNING_FLAGS=" -pedantic"
COMMON_WARNING_FLAGS="${COMMON_WARNING_FLAGS} -Wno-unknown-pragmas"
COMMON_WARNING_FLAGS="${COMMON_WARNING_FLAGS} -Wall"
COMMON_WARNING_FLAGS="${COMMON_WARNING_FLAGS} -Wextra"
COMMON_WARNING_FLAGS="${COMMON_WARNING_FLAGS} -Wformat=2"
COMMON_WARNING_FLAGS="${COMMON_WARNING_FLAGS} -Wredundant-decls"
COMMON_WARNING_FLAGS="${COMMON_WARNING_FLAGS} -Wcast-align"
COMMON_WARNING_FLAGS="${COMMON_WARNING_FLAGS} -Wmissing-declarations"
COMMON_WARNING_FLAGS="${COMMON_WARNING_FLAGS} -Wmissing-include-dirs"
COMMON_WARNING_FLAGS="${COMMON_WARNING_FLAGS} -Wmissing-format-attribute"
COMMON_WARNING_FLAGS="${COMMON_WARNING_FLAGS} -Wswitch-enum"
COMMON_WARNING_FLAGS="${COMMON_WARNING_FLAGS} -Wswitch-default"
COMMON_WARNING_FLAGS="${COMMON_WARNING_FLAGS} -Winvalid-pch"
COMMON_WARNING_FLAGS="${COMMON_WARNING_FLAGS} -Wformat-nonliteral"
COMMON_WARNING_FLAGS="${COMMON_WARNING_FLAGS} -Wno-deprecated-register"
#COMMON_WARNING_FLAGS="${COMMON_WARNING_FLAGS} -flto -Wodr"

#
# Workaround for older compilers warning about { }
# not initializing all struct fields.
#
COMMON_WARNING_FLAGS="${COMMON_WARNING_FLAGS} -Wno-missing-field-initializers"

#
# Don't warn about implicit fallthrough
#
# TODO: Use [[fallthrough]] C++ attribute specifier with
#       newer compiler versions.
#
AX_CHECK_COMPILE_FLAG([-Wno-implicit-fallthrough],
                      [COMMON_WARNING_FLAGS="${COMMON_WARNING_FLAGS} -Wno-implicit-fallthrough"], [], [])

#
# Final project CXXFLAGS are set after configure checks.
#
CXXFLAGS="-std=c++17 $EXTERNAL_CXXFLAGS"
CFLAGS="-std=c99 $EXTERNAL_CFLAGS"
CPPFLAGS="-DHAVE_BUILD_CONFIG_H $EXTERNAL_CPPFLAGS"

#
# Additional CXXFLAGS used when --enable-debug-build is used
#
CXXFLAGS_DEBUG_ENABLED="-g -O0"

#
# Additional CXXFLAGS used when --disable-debug-build is used
# and when --enable-debug-build is
#
# Don't add -DNDEBUG here. It's added to the generated config.h
# header file.
#
CXXFLAGS_DEBUG_DISABLED=""

# Libtool Versioning
# See http://sources.redhat.com/autobook/autobook/autobook_91.html#SEC91 for details

#
# The most recent interface number.
#
# Increment if the interface has additions, changes or removals.
#
LT_CURRENT=1

#
# The implementation number of the current interface.
#
# Increment if library source code changed at all.
# Set to 0 if you increment CURRENT.
#
LT_REVISION=1

#
# The difference between the newest and oldest interfaces
# that the library implements.
#
# Increment if any interfaces have been added.
# Set to 0 if any interfaces have been changed or removed.
# Removal has precedence over adding, so set to 0 if both happened.
#
LT_AGE=0

AC_SUBST(LT_CURRENT)
AC_SUBST(LT_REVISION)
AC_SUBST(LT_AGE)

AC_ARG_ENABLE([coverage],
     [AS_HELP_STRING([--enable-coverage], [enable instrumented compilation for code coverage testing (default=no)])],
     [case "${enableval}" in
       yes) coverage=yes ;;
       no)  coverage=no ;;
       *) AC_MSG_ERROR([bad value ${enableval} for --enable-coverage]) ;;
     esac], [coverage=no])

if test "x$coverage" = xyes; then
   CXXFLAGS="$CXXFLAGS --coverage"
   CFLAGS="$CFLAGS --coverage"
fi

AC_ARG_ENABLE([werror],
     [AS_HELP_STRING([--enable-werror], [treat compiler warnings as errors (default=no)])],
     [case "${enableval}" in
       yes) werror=yes ;;
       no)  werror=no ;;
       *) AC_MSG_ERROR([bad value ${enableval} for --enable-werror]) ;;
     esac], [werror=no])

if test "x$werror" = xyes; then
  COMMON_WARNING_FLAGS=" -Werror $COMMON_WARNING_FLAGS"
fi

#
# Checks for programs.
#
AC_PROG_CXX
AC_PROG_CC_C99
AC_PROG_INSTALL
AC_PROG_MAKE_SET
LT_INIT

# Check if libatomic is available, might be required for emulating
# atomic intrinsics on some platforms.
#
AC_CHECK_LIB([atomic], [__atomic_add_fetch_8], [
    __saved_LIBS="$LIBS"
    LIBS="$LIBS -Wl,--push-state,--as-needed,-latomic,--pop-state"
    AC_LINK_IFELSE([AC_LANG_PROGRAM()],
      [atomic_LIBS="-Wl,--push-state,--as-needed,-latomic,--pop-state"],
      [atomic_LIBS="-latomic"]
    )
    LIBS="$__saved_LIBS"
], [atomic_LIBS=""])
AC_SUBST([atomic_LIBS])

# GNU (or just POSIX) basename(3) function?
AC_MSG_CHECKING([for basename function])
AC_LANG_PUSH([C++])
AC_COMPILE_IFELSE([AC_LANG_SOURCE([
    #define _GNU_SOURCE
    #include <cstring>
    int main(int argc, char ** argv) {
        ::basename(*argv);
        return 0;
    }
])], [
    have_gnu_basename=yes
    AC_MSG_RESULT([GNU])
], [
    have_gnu_basename=no
    AC_MSG_RESULT([POSIX])
])
AC_LANG_POP()
AS_IF([test "x${have_gnu_basename}" = xyes], [
  AC_DEFINE([HAVE_GNU_BASENAME], [1], [Wether the GNU version of function basename(3) is available (or just the POSIX one).])
], [])

# GNU or XSI strerror_r(3) function?
AC_MSG_CHECKING([for strerror_r function])
AC_LANG_PUSH([C++])
AC_COMPILE_IFELSE([AC_LANG_SOURCE([
    #undef _POSIX_C_SOURCE
    #define _GNU_SOURCE
    #include <cstring>
    int main(int argc, char ** argv) {
        char * res = ::strerror_r(0, 0, 0);
        return 0;
    }
])], [
    have_gnu_strerror_r=yes
    AC_MSG_RESULT([GNU])
], [
    have_gnu_strerror_r=no
    AC_MSG_RESULT([XSI])
])
AC_LANG_POP()
AS_IF([test "x${have_gnu_strerror_r}" = xyes], [
  AC_DEFINE([HAVE_GNU_STRERROR_R], [1], [Wether the GNU version of function strerror_r(3) is available (or just the XSI one).])
], [])

#
# Checks for required libraries.
#
AX_PTHREAD
AC_MSG_CHECKING([for NETLINK_KOBJECT_UEVENT definition])
AC_COMPILE_IFELSE([AC_LANG_SOURCE([[
#include <linux/netlink.h>
int main(void)
{
  return NETLINK_KOBJECT_UEVENT;
}
]])],
[have_uevent=yes], [have_uevent=no])
if test "x$have_uevent" = xyes; then
  AC_MSG_RESULT([yes])
  AC_DEFINE([HAVE_UEVENT], [1], [UEVENT netlink socket support])
else
  AC_MSG_RESULT([no])
fi

#
# qb library
#
PKG_CHECK_MODULES([qb], [libqb >= 0.16.0],
[AC_DEFINE([HAVE_LIBQB], [1], [libqb API available])
libqb_summary="system-wide; $qb_CFLAGS $qb_LIBS"],
[AC_MSG_FAILURE([libqb development files not found!])]
)

#
# sodium library
#
PKG_CHECK_MODULES([sodium], [libsodium >= 0.4.5],
[AC_DEFINE([HAVE_LIBSODIUM], [1], [libsodium API available])
libsodium_summary="system-wide; $sodium_CFLAGS $sodium_LIBS"]
libsodium_available=yes,
[]
)

#
# libcrypto library (OpenSSL)
#
PKG_CHECK_MODULES([libcrypto], [libcrypto >= 1.0.0],
[AC_DEFINE([HAVE_LIBCRYPTO], [1], [libcrypto API available])
libcrypto_summary="system-wide; $libcrypto_CFLAGS $libcrypto_LIBS"]
libcrypto_available=yes,
[]
)

#
# gcrypt library
#
AM_PATH_LIBGCRYPT([1.5.0],
[AC_DEFINE([HAVE_LIBGCRYPT], [1], [libgcrypt API available])
gcrypt_CFLAGS="$LIBGCRYPT_CFLAGS"
gcrypt_LIBS="$LIBGCRYPT_LIBS"
libgcrypt_summary="system-wide; $gcrypt_CFLAGS $gcrypt_LIBS"
libgcrypt_available=yes],
[]
)

#
# --with-crypto-library=<selector>
#
# Supported selectors:
#
#  sodium ... libsodium
#  gcrypt ... libgcrypt
#  openssl ... libcrypto
#
AC_ARG_WITH([crypto-library], AS_HELP_STRING([--with-crypto-library],
            [Select crypto backend library. Supported values: sodium, gcrypt, openssl.]),
            [with_crypto_library=$withval], [with_crypto_library=sodium])

case "$with_crypto_library" in
  openssl)
    if test "x$libcrypto_available" = xyes; then
      crypto_CFLAGS="$libcrypto_CFLAGS"
      crypto_LIBS="$libcrypto_LIBS"
      crypto_summary="$libcrypto_summary"
      AC_DEFINE([USBGUARD_USE_OPENSSL], [1], [Use openssl as crypto backend])
     else
      AC_MSG_ERROR([The selected crypto backend library is not available.])
    fi
  ;;
  sodium)
    if test "x$libsodium_available" = xyes; then
      crypto_CFLAGS="$sodium_CFLAGS"
      crypto_LIBS="$sodium_LIBS"
      crypto_summary="$libsodium_summary"
      AC_DEFINE([USBGUARD_USE_LIBSODIUM], [1], [Use libsodium as crypto backend])
     else
      AC_MSG_ERROR([The selected crypto backend library is not available.])
    fi
  ;;
  gcrypt)
    if test "x$libgcrypt_available" = xyes; then
      crypto_CFLAGS="$gcrypt_CFLAGS"
      crypto_LIBS="$gcrypt_LIBS"
      crypto_summary="$libgcrypt_summary"
      AC_DEFINE([USBGUARD_USE_LIBGCRYPT], [1], [Use libsodium as crypto backend])
    else
      AC_MSG_ERROR([The selected crypto backend library is not available.])
    fi
  ;;
  *)
  AC_MSG_FAILURE([Invalid crypto library selector. Supported selectors: sodium, gcrypt, openssl])
esac
AC_SUBST([crypto_CFLAGS])
AC_SUBST([crypto_LIBS])

#
# Check for optional libraries
#
AC_ARG_ENABLE([audit],
              [AS_HELP_STRING([--enable-audit], [Enables Linux Audit support if available (default=yes)])],
              [use_audit=$enableval], [use_audit=yes])

if test "x$use_audit" = xyes; then
  PKG_CHECK_MODULES([audit], [audit >= 2.7.7],
  [AC_DEFINE([HAVE_LINUX_AUDIT], [1], [Linux Audit API available])
  AC_CHECK_DECLS([audit_encode_nv_string], [], [], [[#include<libaudit.h>]])
  libaudit_summary="system-wide; $audit_LIBS"],
  [AC_MSG_NOTICE([libaudit development files not found! Linux Audit support wille be turned OFF])
  libaudit_summary="not found"]
  )
else
  libaudit_summary="not enabled"
fi

AC_ARG_ENABLE([seccomp],
              [AS_HELP_STRING([--enable-seccomp], [Enables Seccomp support if available (default=yes)])],
              [use_seccomp=$enableval], [use_seccomp=yes])

if test "x$use_seccomp" = xyes; then
  PKG_CHECK_MODULES([seccomp], [libseccomp >= 2.0.0],
  [AC_DEFINE([HAVE_SECCOMP], [1], [seccomp API usable])
  libseccomp_summary="system-wide; $seccomp_LIBS"],
  [AC_MSG_NOTICE([libseccomp development files not found! Seccomp support will be turned OFF])
  libseccomp_summary="not found"]
  )
else
  libseccomp_summary="not enabled"
fi

AC_ARG_ENABLE([libcapng],
              [AS_HELP_STRING([--enable-libcapng], [Enables POSIX 1003.1e capability support if available (default=yes)])],
              [use_libcapng=$enableval], [use_libcapng=yes])

if test "x$use_libcapng" = xyes; then
  PKG_CHECK_MODULES([libcapng], [libcap-ng >= 0.7.0],
  [AC_DEFINE([HAVE_LIBCAPNG], [1], [cap-ng API usable])
  libcap_ng_summary="system-wide; $libcapng_LIBS"],
  [AC_MSG_NOTICE([libcap-ng development files not found! Support for POSIX 1003.1e capabilities will be turned OFF])
  libcap_ng_summary="not found"]
  )
else
  libcap_ng_summary="not enabled"
fi

AC_ARG_ENABLE([umockdev],
              [AS_HELP_STRING([--enable-umockdev], [Enables umockdev support if available (default=yes)])],
              [use_umockdev=$enableval], [use_umockdev=yes])

if test "x$use_umockdev" = xyes; then
  PKG_CHECK_MODULES([umockdev], [umockdev-1.0 >= 0.8.0],
  [AC_DEFINE([HAVE_UMOCKDEV], [1], [umockdev API usable])
  umockdev_summary="system-wide; $umockdev_LIBS"
  umockdev_available=yes],
  [AC_MSG_NOTICE([umockdev development files not found! umockdev device manager won't be available])
  umockdev_summary="not found"]
  )
else
  umockdev_summary="not enabled"
fi

PKG_CHECK_MODULES([protobuf], [protobuf >= 2.5.0],
[protobuf_summary="system-wide; $protobuf_CFLAGS $protobuf_LIBS"],
[AC_MSG_ERROR([Required protobuf development files not found!])]
)

AC_CHECK_PROGS(PROTOC, [protoc])
if test -z "$PROTOC"; then
  AC_MSG_ERROR(["Required protobuf compiler not found!"])
fi

#
# Catch C++ library
#
AC_ARG_ENABLE([catch],
  [AS_HELP_STRING([--enable-catch], [Enable Catch testing framework support (default=yes)])],
  [enable_catch=$enableval], [enable_catch=yes])

AC_ARG_WITH([bundled-catch], AS_HELP_STRING([--with-bundled-catch], [Build using the bundled Catch library]), [with_bundled_catch=$withval], [with_bundled_catch=no])

if test "x$enable_catch" = xyes; then
  if test "x$with_bundled_catch" = xyes; then
    catch_CFLAGS="-I\$(top_srcdir)/src/ThirdParty/Catch/single_include/catch2"
    catch_LIBS=""
    AC_MSG_NOTICE([Using bundled Catch library])
    catch_summary="bundled; $catch_CFLAGS $catch_LIBS"
  else
    SAVE_CPPFLAGS=$CPPFLAGS
    CPPFLAGS="-std=c++17 $CPPFLAGS -I/usr/include/catch2"
    AC_LANG_PUSH([C++])
    AC_CHECK_HEADER([catch_test_macros.hpp],
    [catch_CFLAGS="-I/usr/include/catch2 -DHAVE_CATCH2_V3"
    catch_LIBS="-lCatch2Main -lCatch2"],
    [AC_CHECK_HEADER([catch.hpp],
    [catch_CFLAGS="-I/usr/include/catch2"
    catch_LIBS=""],
    [AC_MSG_FAILURE(Catch2 not found or not usable. Re-run with --with-bundled-catch to use the bundled library.)]
    )])
    AC_LANG_POP
    CPPFLAGS=$SAVE_CPPFLAGS
    catch_summary="system-wide; $catch_CFLAGS $catch_LIBS"
  fi
else
  catch_CFLAGS=""
  catch_LIBS=""
  catch_summary="disabled; not checking for Catch2"
fi
AC_SUBST([catch_CFLAGS])
AC_SUBST([catch_LIBS])

#
# stdc++fs (for PEGTL >=3 below, e.g. with GCC 8)
#
STDCXX_FS_LINKTEST_FILENAME="${srcdir}"/src/test_filesystem.cpp
STDCXX_FS_LINKTEST_SOURCE="$(cat "${STDCXX_FS_LINKTEST_FILENAME}")"
AC_LANG_PUSH([C++])
AC_MSG_CHECKING([whether we need to link to -lstdc++fs for PEGTL explicitly])
AC_LINK_IFELSE([AC_LANG_SOURCE([${STDCXX_FS_LINKTEST_SOURCE}])], [
    AC_MSG_RESULT([no])
    stdcxxfs_LIBS=''
], [
    stdcxxfs_LIBS='-lstdc++fs'
    SAVE_LIBS=${LIBS}
    LIBS="${LIBS} ${stdcxxfs_LIBS}"
    AC_LINK_IFELSE([AC_LANG_SOURCE([${STDCXX_FS_LINKTEST_SOURCE}])], [
        AC_MSG_RESULT([yes])
    ], [
        AC_MSG_RESULT([ERROR])
        AC_MSG_ERROR([Link test failed both with and without ${stdcxxfs_LIBS}; something is broken, please check file config.log for details.])
    ])
    LIBS=${SAVE_LIBS}
])
AC_LANG_POP()

#
# PEGTL C++ library
#
AC_ARG_WITH([bundled-pegtl], AS_HELP_STRING([--with-bundled-pegtl], [Build using the bundled PEGTL library]), [with_bundled_pegtl=$withval], [with_bundled_pegtl=no])
if test "x$with_bundled_pegtl" = xyes; then
	pegtl_CFLAGS="-I\$(top_srcdir)/src/ThirdParty/PEGTL/include"
	pegtl_AC_CFLAGS="-I$srcdir/src/ThirdParty/PEGTL/include"
	pegtl_LIBS="${stdcxxfs_LIBS}"
	AC_MSG_NOTICE([Using bundled PEGTL library])
	pegtl_summary="bundled; $pegtl_CFLAGS $pegtl_LIBS"
else
	pegtl_CFLAGS=""
	pegtl_AC_CFLAGS=""
	pegtl_LIBS="${stdcxxfs_LIBS}"
	pegtl_summary="system-wide; $pegtl_CFLAGS $pegtl_LIBS"
fi
AC_SUBST([pegtl_CFLAGS])
AC_SUBST([pegtl_AC_CFLAGS])
AC_SUBST([pegtl_LIBS])

SAVE_CPPFLAGS=$CPPFLAGS
CPPFLAGS="-std=c++17 $CPPFLAGS $pegtl_AC_CFLAGS"
AC_LANG_PUSH([C++])
AC_CHECK_HEADER([tao/pegtl.hpp],
		[AC_DEFINE([HAVE_TAO_PEGTL_HPP], [1], [PEGTL header file with .hpp extension is present])],
		[AC_MSG_FAILURE(PEGTL header file not found or not usable. Re-run with --with-bundled-pegtl to use the bundled library.)])
AC_LANG_POP
CPPFLAGS=$SAVE_CPPFLAGS

#
# GDBus
#
AC_ARG_WITH([dbus], AS_HELP_STRING([--with-dbus], [Build the DBus Bridge service]), [], [with_dbus=yes])
if test "x$with_dbus" = xyes; then
  #
  # Check for required D-Bus modules
  #
  PKG_CHECK_MODULES([dbus], [dbus-1 gio-2.0 >= 2.46 polkit-gobject-1],
  [AC_DEFINE([HAVE_DBUS], [1], [Required GDBus API available])
  dbus_summary="system-wide; $dbus_CFLAGS $dbus_LIBS"],
  [AC_MSG_FAILURE([Required D-Bus modules (dbus-1, gio-2.0, polkit-gobject-1) not found!])]
  )

  #
  # Check for xmllint tool
  #
  AC_CHECK_PROGS(XMLLINT, [xmllint])
  if test -z "$XMLLINT" ; then
    AC_MSG_FAILURE([The required xmllint tool is missing on your system. Make sure that xmllint is installed.])
  fi
  #
  # Check for xsltproc tool
  #
  AC_CHECK_PROGS(XSLTPROC, [xsltproc])
  if test -z "$XSLTPROC"; then
    AC_MSG_FAILURE([The required xsltproc tool is missing on your system. Make sure that xsltproc is installed.])
  fi
  #
  # Check for gdbus-codegen tool
  #
  AC_CHECK_PROGS(GDBUS_CODEGEN, [gdbus-codegen])
  if test -z "$GDBUS_CODEGEN"; then
    AC_MSG_FAILURE([The required gdbus-codegen tool is missing on your system. Make sure that gdbus-codegen is installed.])
  fi

  dbus_services_dir=`$PKG_CONFIG dbus-1 --define-variable=datadir='${datadir}' --define-variable=prefix='${prefix}' --variable=system_bus_services_dir`
  if test "x$dbus_services_dir" = x; then
    AC_MSG_FAILURE([Cannot autodetect the D-Bus system services directory])
  else
    #
    # Hack around expanded paths in system_bus_services_dir. Without this distcheck won't
    # work, because it uses ./configure --prefix=$somedir. The expanded path ignores this
    # prefix (and DESTDIR won't help here either)
    #
    case "$dbus_services_dir" in
      /usr/share/*|/usr/local/share/*)
        dbus_services_dir=$(echo "$dbus_services_dir" | sed -r 's,^(/usr/share|/usr/local/share),${datadir},')
      ;;
      /usr/*|/usr/local/*)
        dbus_services_dir=$(echo "$dbus_services_dir" | sed -r 's,^(/usr|/usr/local),${prefix},')
      ;;
      /*)
        dbus_services_dir='${prefix}'"$dbus_services_dir"
      ;;
    esac
    AC_SUBST(DBUS_SERVICES_DIR, $dbus_services_dir)
  fi

  dbus_data_dir=`$PKG_CONFIG dbus-1 --define-variable=datadir='${datadir}' --define-variable=prefix='${prefix}' --variable=datadir`
  if test "x$dbus_data_dir" = x; then
    AC_MSG_FAILURE([Cannot autodetect the D-Bus data directory])
  else
    dbus_busconfig_dir="$dbus_data_dir/dbus-1/system.d"
    AC_SUBST(DBUS_BUSCONFIG_DIR, $dbus_busconfig_dir)
  fi
else
  dbus_summary="None; building without DBus support"
  dbus_services_dir="-"
  dbus_busconfig_dir="-"
fi

#
# PolicyKit
#
AC_ARG_WITH([polkit], AS_HELP_STRING([--with-polkit], [Install the PolicyKit configuration if D-Bus support is also enabled]), [], [with_polkit=yes])
if test "x$with_polkit" = xyes; then
  #
  # Check for required PolicyKit modules
  #
  PKG_CHECK_MODULES([polkit], [polkit-gobject-1],
  [AC_DEFINE([HAVE_POLKIT], [1], [Required PolicyKit modules available])
  polkit_summary="system-wide; `$PKG_CONFIG polkit-gobject-1 --define-variable=prefix='${prefix}' --define-variable=datadir='${datadir}' --variable=policydir`"],
  [AC_MSG_FAILURE([Required PolicyKit modules (polkit-gobject-1) not found!])]
  )

  polkit_policy_dir="`$PKG_CONFIG polkit-gobject-1 --define-variable=prefix='${prefix}' --define-variable=datadir='${datadir}' --variable=policydir`"
  if test "x$polkit_policy_dir" = x; then
    AC_MSG_FAILURE([Cannot autodetect the PolicyKit policy directory])
  else
    AC_SUBST(POLKIT_POLICY_DIR, $polkit_policy_dir)
  fi
else
  polkit_summary="None; building without PolicyKit support"
  polkit_policy_dir="-"
fi

#
# LDAP
#
AC_ARG_WITH([ldap], AS_HELP_STRING([--with-ldap], [Build USBGuard with ldap support]), [], [with_ldap=no])
if test "x$with_ldap" = xyes; then
  #
  # Check for LDAP ldap.h
  #
  AC_CHECK_HEADER(
      [ldap.h],
      [
        ldap_LIBS="-lldap"
        ldap_CFLAGS=""
        AC_DEFINE([HAVE_LDAP], [1], [ldap API available])
        libldap_summary="system-wide; $ldap_CFLAGS $ldap_LIBS"
      ],
      [
        ldap_LIBS=""
        ldap_CFLAGS=""
        AC_MSG_ERROR([Building with --with-ldap but ldap.h is not available])
        libldap_summary="not found"
      ]
)
else
  libldap_summary="None; building without LDAP support"
fi
AC_SUBST(ldap_LIBS)
AC_SUBST(ldap_CFLAGS)

#
# AddressSanitizer
#
# Based on: https://github.com/machinezone/tcpkali/blob/15903a7692abecbd28899d44e822a6f894caa23c/configure.aci#L73
#
# Enable Address Sanitizer, if supported by gcc (4.8+) or clang.
# http://clang.llvm.org/docs/AddressSanitizer.html
# https://code.google.com/p/address-sanitizer/wiki/HowToBuild
#
AC_ARG_ENABLE([asan],
  [AS_HELP_STRING([--enable-asan], [Enable Address Sanitizer])],
  [enable_asan=$enableval], [enable_asan=no])

if test "x$enable_asan" = xyes; then
  AC_LANG_PUSH([C++])
  AX_CHECK_COMPILE_FLAG([-faddress-sanitizer],
                        [ASAN_FLAGS="$ASAN_FLAGS -faddress-sanitizer"],
                        [enable_asan=no], [])
  if test "x$enable_asan" = xno; then
    AX_CHECK_COMPILE_FLAG([-fsanitize=address],
                          [ASAN_FLAGS="$ASAN_FLAGS -fsanitize=address"
                           enable_asan=yes],
                          [enable_asan=no], [])
    if test "x$enable_asan" = xno; then
      AC_MSG_FAILURE([--enable-asan is given, but not supported on this platform. Check out https://code.google.com/p/address-sanitizer/wiki/HowToBuild])
    fi
  fi
  # Keep error messages nice. Also consider:
  # export ASAN_SYMBOLIZER_PATH=/usr/bin/llvm-symbolizer
  # export ASAN_OPTIONS=symbolize=1
  AX_CHECK_COMPILE_FLAG([-fno-omit-frame-pointer],
                        [ASAN_FLAGS="$ASAN_FLAGS -fno-omit-frame-pointer"],
                        [], [])
  AX_CHECK_COMPILE_FLAG([-static-libasan],
                        [ASAN_FLAGS="$ASAN_FLAGS -static-libasan"],
                        [], [])
  AC_LANG_POP
fi
#
# Enable Thread Sanitizer, if supported by clang.
# http://clang.llvm.org/docs/ThreadSanitizer.html
# https://code.google.com/p/address-sanitizer/wiki/HowToBuild
#
AC_ARG_ENABLE([tsan],
    [AS_HELP_STRING([--enable-tsan], [Enable Thread Sanitizer])],
    [enable_tsan=$enableval], [enable_tsan=no])

if test "x$enable_tsan" = xyes; then
  if test "x$enable_asan" = xyes; then
    AC_MSG_FAILURE([Address Sanitizer (ASAN) and Thread Sanitizer (TSAN) are mutually exclusive])
  fi
  AX_CHECK_COMPILE_FLAG([-fsanitize=thread],
                        [TSAN_FLAGS="$TSAN_FLAGS -fsanitize=thread"],
                        [enable_tsan=no], [])
  if test "x$enable_tsan" = xno; then
    AC_MSG_FAILURE([--enable-tsan is given, but not supported on this platform. Check out https://code.google.com/p/address-sanitizer/wiki/HowToBuild])
  fi
  # Keep error messages nice.
  AX_CHECK_COMPILE_FLAG([-fno-omit-frame-pointer],
                        [TSAN_FLAGS="$TSAN_FLAGS -fno-omit-frame-pointer"],
                        [], [])
  # -fsanitize=thread linking must be done with -pie or -shared
  # We can't do anything about this message in GCC; use clang.
fi

#
# Check whether asciidoc (a2x) is present.
#
AC_CHECK_PROGS(A2X, [a2x])
if test -z "$A2X"; then
  AC_MSG_WARN([Unable to find documentation generator (asciidoc / a2x). Documentation will not be generated.])
fi

#
# Check whether the aspell utility is present.
#
# If aspell is not present, the test suite will skip spell
# checking of documentation related files.
#
AC_CHECK_PROGS(ASPELL, [aspell])
if test -z "$ASPELL"; then
  AC_MSG_WARN([Unable to find the Aspell spell checker. The test suite will skip spell checking of documentation related files.])
fi

#
# Full test suite mode.
#
# Runs all available test when enabled. Some tests make assumption about
# the environment they run in, so they might fail when these assumptions
# are not met.
#
AC_ARG_ENABLE([full-test-suite],
              [AS_HELP_STRING([--enable-full-test-suite], [Run the full test suite (default=no)])],
              [full_test_suite=$enableval], [full_test_suite=no])

if test "x$full_test_suite" = xyes; then
  if test "x$umockdev_available" != xyes; then
    AC_MSG_ERROR([umockdev is required to run the full test suite.])
  fi
fi

# Checks for header files.
AC_LANG_PUSH([C++])
AC_FUNC_ALLOCA
AC_CHECK_HEADERS([arpa/inet.h fcntl.h inttypes.h limits.hi locale.h netdb.h stdint.h stdlib.h])
AC_CHECK_HEADERS([string.h sys/time.h syslog.h unistd.h wchar.h ext/stdio_filebuf.h])

# Checks for typedefs, structures, and compiler characteristics.
AC_CHECK_HEADER_STDBOOL
AC_C_INLINE
AC_TYPE_INT16_T
AC_TYPE_INT32_T
AC_TYPE_INT64_T
AC_TYPE_INT8_T
AC_TYPE_PID_T
AC_TYPE_SIZE_T
AC_TYPE_SSIZE_T
AC_TYPE_UID_T
AC_TYPE_UINT16_T
AC_TYPE_UINT32_T
AC_TYPE_UINT64_T
AC_TYPE_UINT8_T

# Checks for library functions.
AC_FUNC_FORK
AC_FUNC_LSTAT_FOLLOWS_SLASHED_SYMLINK
AC_FUNC_MALLOC
AC_FUNC_REALLOC
AC_CHECK_FUNCS([dup2 gettimeofday localtime_r memset mkdir regcomp setlocale strerror strstr])
AC_LANG_POP

# ./configure script options
AC_ARG_ENABLE([debug-build],
     [AS_HELP_STRING([--enable-debug-build], [enable debugging flags (default=no)])],
     [case "${enableval}" in
       yes) debug=yes ;;
       no)  debug=no ;;
       *) AC_MSG_ERROR([bad value ${enableval} for --enable-debug-build]) ;;
     esac], [debug=no])

AC_ARG_ENABLE([systemd],
     [AS_HELP_STRING([--enable-systemd], [install the systemd service unit file (default=no)])],
     [case "${enableval}" in
       yes) systemd=yes ;;
       no)  systemd=no ;;
       *) AC_MSG_ERROR([bad value ${enableval} for --enable-systemd]) ;;
     esac], [systemd=no])

AC_ARG_WITH([bash-completion-dir],
	AS_HELP_STRING([--with-bash-completion-dir[=PATH]],
		[Enable bash auto-completion. Uses pkgconfig if no path given. @<:@default=yes@:>@]),
	[], [with_bash_completion_dir=yes])

if test "x$with_bash_completion_dir" = "xyes"; then
	PKG_CHECK_MODULES([BASH_COMPLETION], [bash-completion >= 2.0],
		[BASH_COMPLETION_DIR=$($PKG_CONFIG --variable=completionsdir bash-completion)],
		[BASH_COMPLETION_DIR="$datadir/bash-completion/completions"])
else
	BASH_COMPLETION_DIR="$with_bash_completion_dir"
fi

if test "x$debug" = xyes; then
   CXXFLAGS="$CXXFLAGS $CXXFLAGS_DEBUG_ENABLED"
   CFLAGS="$CFLAGS $CFLAGS_DEBUG_ENABLED"
else
   CXXFLAGS="$CXXFLAGS $CXXFLAGS_DEBUG_DISABLED"
   CFLAGS="$CFLAGS $CFLAGS_DEBUG_DISABLED"
   AC_DEFINE([NDEBUG], [1], [Debug build disabled])
   AC_DEFINE([QUEX_OPTION_ASSERTS_DISABLED], [1], [Disable asserts in the Quex generated lexer])
fi

if test "x$enable_asan" = xyes; then
  CXXFLAGS="$CXXFLAGS $ASAN_FLAGS"
  CFLAGS="$CFLAGS $ASAN_FLAGS"
  LDFLAGS="$LDFLAGS $ASAN_FLAGS"
  AC_DEFINE([ASAN_ENABLED], [1], [Address Sanitizer is enabled])
fi

if test "x$enable_tsan" = xyes; then
  CXXFLAGS="$CXXFLAGS $TSAN_FLAGS"
  CFLAGS="$CFLAGS $TSAN_FLAGS"
  LDFLAGS="$LDFLAGS $TSAN_FLAGS"
  AC_DEFINE([TSAN_ENABLED], [1], [Thread Sanitizer is enabled])
fi

if test "x$systemd" = xyes; then
   systemd_unit_dir="`$PKG_CONFIG systemd --variable=systemdsystemunitdir`"
   if test "x$systemd_unit_dir" = x; then
     AC_MSG_FAILURE([Cannot detect the systemd system unit dir])
   fi
   AC_SUBST(SYSTEMD_UNIT_DIR, $systemd_unit_dir)
   AC_DEFINE([SYSTEMD_SUPPORT_ENABLED], [1], [Enable systemd support int the project])
else
  systemd_unit_dir="-"
fi

AC_SUBST([ANALYZE_CONFIGURE_ARGS], $ac_configure_args)

AC_SUBST([BASH_COMPLETION_DIR])
AM_CONDITIONAL([ENABLE_BASH_COMPLETION], [test "x$with_bash_completion_dir" != "xno"])

AM_CONDITIONAL([SYSTEMD_SUPPORT_ENABLED], [test "x$systemd" = xyes ])
AM_CONDITIONAL([DBUS_ENABLED], [test "x$with_dbus" = xyes ])
AM_CONDITIONAL([DOCS_ENABLED], [! test -z "$A2X" ])
AM_CONDITIONAL([POLICYKIT_ENABLED], [test "x$with_polkit" = xyes])
AM_CONDITIONAL([FULL_TEST_SUITE_ENABLED], [test "x$full_test_suite" = xyes])
AM_CONDITIONAL([WITH_LDAP], [test "x$with_ldap" = xyes])
AM_CONDITIONAL([BASH_COMPLETION_ENABLED], [test "x$bash_completion" != xno])
AM_CONDITIONAL([CATCH_ENABLED], [test "x$enable_catch" = xyes ])

CXXFLAGS="$CXXFLAGS -fvisibility=hidden $COMMON_WARNING_FLAGS $WARNING_CXXFLAGS"
CFLAGS="$CFLAGS -fvisibility=hidden $COMMON_WARNING_FLAGS $WARNING_CFLAGS"

AC_DEFINE_UNQUOTED([BINDIR], ["$bindir"], [Binaries root directory])
AC_DEFINE_UNQUOTED([DATADIR], ["$datadir"], [Data directory])
AC_DEFINE_UNQUOTED([DATAROOTDIR], ["$datarootdir"], [Data root directory])
AC_DEFINE_UNQUOTED([DOCDIR], ["$docdir"], [Documentation directory])
AC_DEFINE_UNQUOTED([EXEC_PREFIX], ["$exec_prefix"], [Exec Prefix])
AC_DEFINE_UNQUOTED([INCLUDEDIR], ["$includedir"], [Include directory])
AC_DEFINE_UNQUOTED([LIBEXECDIR], ["$libexecdir"], [Libexec directory])
AC_DEFINE_UNQUOTED([LOCALSTATEDIR], ["$localstatedir"], [Local state directory])
AC_DEFINE_UNQUOTED([PREFIX], ["$prefix"], [Prefix])
AC_DEFINE_UNQUOTED([SBINDIR], ["$sbindir"], [System binaries root directory])
AC_DEFINE_UNQUOTED([SHAREDSTATEDIR], ["$sharedstatedir"], [Shared state directory])
AC_DEFINE_UNQUOTED([SYSCONFDIR], ["$sysconfdir"], [System config root directory])

AC_DEFINE_UNQUOTED([USBGUARD_DAEMON_CONF_PATH], ["$sysconfdir/usbguard/usbguard-daemon.conf"], [Default USBGuard daemon config path])

AC_CONFIG_FILES([Makefile
                 libusbguard.pc
                 src/Tests/Fuzzers/Makefile
                 src/Tests/Makefile])
AC_OUTPUT

echo
echo "    Build Configuration Summary    "
echo "==================================="
echo
echo " Run full test suite: $full_test_suite"
echo
echo "## Libraries"
echo
echo "      libqb: $libqb_summary"
echo "     crypto: $crypto_summary"
echo "   libaudit: $libaudit_summary"
echo "    libldap: $libldap_summary"
echo " libseccomp: $libseccomp_summary"
echo "  libcap-ng: $libcap_ng_summary"
echo "   protobuf: $protobuf_summary"
echo "      Catch: $catch_summary"
echo "      PEGTL: $pegtl_summary"
echo "      GDBus: $dbus_summary"
echo "   umockdev: $umockdev_summary"
echo
echo "## Directories"
echo
echo " D-Bus System Services: $dbus_services_dir"
echo "       D-Bus Busconfig: $dbus_busconfig_dir"
echo "    PolicyKit Policies: $polkit_policy_dir"
echo "      systemd unit dir: $systemd_unit_dir"
echo "   Bash completion dir: $BASH_COMPLETION_DIR"
echo
echo "## Compilation Flags"
echo
echo "Debug Build: $debug"
echo "       DEFS: $DEFS"
echo "   CXXFLAGS: $CXXFLAGS"
echo "     CFLAGS: $CFLAGS"
echo "   CPPFLAGS: $CPPFLAGS"
echo "    LDFLAGS: $LDFLAGS"
echo " ASAN_FLAGS: $ASAN_FLAGS"
echo " TSAN_FLAGS: $TSAN_FLAGS"
echo
