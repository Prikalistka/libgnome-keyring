AC_INIT([libgnome-keyring], [3.10.0])
AC_CONFIG_SRCDIR([library/gnome-keyring.h])
AM_INIT_AUTOMAKE([tar-ustar])
AC_CONFIG_MACRO_DIR([build/m4])

AC_CONFIG_HEADERS(config.h)
m4_ifdef([AM_SILENT_RULES], [AM_SILENT_RULES([yes])])

AM_SANITY_CHECK
AM_MAINTAINER_MODE([enable])

AC_GNU_SOURCE
AC_ISC_POSIX
AC_PROG_CC
AM_PROG_CC_C_O
AC_PROG_INSTALL
AC_PROG_LN_S
AC_PROG_MAKE_SET
AM_DISABLE_STATIC
AM_PROG_LIBTOOL
IT_PROG_INTLTOOL([0.35.0])
AC_PATH_PROG(PKG_CONFIG, pkg-config, no)

LIB_GNOME_KEYRING_LT_VERSION=2:0:2
AC_SUBST(LIB_GNOME_KEYRING_LT_VERSION)

version_bits=`echo $VERSION | tr '.' ' '`
full_version=`printf %03d%03d%03d $version_bits`
AC_DEFINE_UNQUOTED(GKR_VERSION, $full_version, [The libgnome-keyring version as a number])

AC_TYPE_PID_T
AC_TYPE_SIGNAL
AC_TYPE_SIZE_T
AC_TYPE_UID_T

GTK_DOC_CHECK(1.9)
DISTCHECK_CONFIGURE_FLAGS="--enable-gtk-doc"
AC_SUBST(DISTCHECK_CONFIGURE_FLAGS)

PKG_CHECK_MODULES(GLIB, glib-2.0 >= 2.16.0 gobject-2.0 gmodule-2.0)
AC_SUBST(GLIB_CFLAGS)
AC_SUBST(GLIB_LIBS)

LIBRARY_CFLAGS="$LIBRARY_CFLAGS $GLIB_CFLAGS"
LIBRARY_LIBS="$LIBRARY_LIBS $GLIB_LIBS"

GETTEXT_PACKAGE="libgnome-keyring"
AC_DEFINE_UNQUOTED(GETTEXT_PACKAGE, "$GETTEXT_PACKAGE", [The gettext domain name])
AC_SUBST(GETTEXT_PACKAGE)

AM_GLIB_GNU_GETTEXT

# Introspection
GOBJECT_INTROSPECTION_CHECK([1.30.0])

# Vala bindings
VAPIGEN_CHECK([0.16])

AC_ARG_WITH(vapidir, [AC_HELP_STRING([--with-vapidir], [Directory for VAPI files]),
            ], [VAPIGEN_VAPIDIR=$with_vapidir])

if test "$with_vapidir" = ""; then
	VAPIDIR="$VAPIGEN_VAPIDIR"
else
	VAPIDIR="$with_vapidir"
fi

AC_SUBST(VAPIDIR)

# --------------------------------------------------------------------
# Check for various credentials.
#

AC_MSG_CHECKING(for struct cmsgcred)
AC_TRY_COMPILE([
#include <sys/types.h>
#include <sys/socket.h>
],[
struct cmsgcred cred;

cred.cmcred_pid = 0;
],have_struct_cmsgcred=yes,have_struct_cmsgcred=no)
AC_MSG_RESULT($have_struct_cmsgcred)

if test x$have_struct_cmsgcred = xyes; then
	AC_DEFINE(HAVE_CMSGCRED, 1, [Have cmsgcred structure])
fi

# --------------------------------------------------------------------
# Check for getpeerucred
#

AC_CHECK_FUNCS(getpeerucred, AC_DEFINE(HAVE_GETPEERUCRED,1,[Have getpeerucred]))

# --------------------------------------------------------------------
# Check for getpeereid
#

AC_CHECK_FUNCS(getpeereid, AC_DEFINE(HAVE_GETPEEREID, 1, [Have getpeereid function]))

# --------------------------------------------------------------------
# Memory locking
#

AC_CHECK_FUNCS(mlock)

# --------------------------------------------------------------------
# DBus Checks
#

DBUS_REQUIRED=1.0
PKG_CHECK_MODULES([DBUS], dbus-1 >= 1.0)

DBUS_VERSION=`$PKG_CONFIG --modversion dbus-1`
DBUS_MAJOR_VERSION=`echo $DBUS_VERSION | cut -d. -f1`
DBUS_MINOR_VERSION=`echo $DBUS_VERSION | cut -d. -f2`
DBUS_MICRO_VERSION=`echo $DBUS_VERSION | cut -d. -f3`
DBUS_CFLAGS="$DBUS_CFLAGS -DGKR_DBUS_MAJOR_VERSION=$DBUS_MAJOR_VERSION"
DBUS_CFLAGS="$DBUS_CFLAGS -DGKR_DBUS_MINOR_VERSION=$DBUS_MINOR_VERSION"
DBUS_CFLAGS="$DBUS_CFLAGS -DGKR_DBUS_MICRO_VERSION=$DBUS_MICRO_VERSION"

LIBRARY_CFLAGS="$LIBRARY_CFLAGS $DBUS_CFLAGS"
LIBRARY_LIBS="$LIBRARY_LIBS $DBUS_LIBS"

# --------------------------------------------------------------------
# libgcrypt
#

GCRYPT_VERSION=1.2.2
GCRYPT_LIBVER=1

AM_PATH_LIBGCRYPT($GCRYPT_LIBVER:$GCRYPT_VERSION,,
	AC_MSG_ERROR([[
***
*** libgcrypt was not found. You may want to get it from
*** ftp://ftp.gnupg.org/gcrypt/libgcrypt/
***
]]))
libgcrypt=yes
AC_DEFINE_UNQUOTED(LIBGCRYPT_VERSION, "$GCRYPT_VERSION", [Version of GCRYPT we expect])

LIBRARY_CFLAGS="$LIBRARY_CFLAGS $LIBGCRYPT_CFLAGS"
LIBRARY_LIBS="$LIBRARY_LIBS $LIBGCRYPT_LIBS"

AC_SUBST([LIBGCRYPT_CFLAGS])
AC_SUBST([LIBGCRYPT_LIBS])

# --------------------------------------------------------------------
# Debug mode
#

AC_MSG_CHECKING([for debug mode])
AC_ARG_ENABLE(debug,
              AC_HELP_STRING([--enable-debug=no/default/yes],
              [Turn on or off debugging]))

if test "$enable_debug" != "no"; then
	AC_DEFINE_UNQUOTED(WITH_DEBUG, 1, [Print debug output])
	AC_DEFINE_UNQUOTED(_DEBUG, 1, [In debug mode])
	CFLAGS="$CFLAGS -g"
fi

if test "$enable_debug" = "yes"; then
	debug_status="yes (-g, -O0, debug output, testable)"
	CFLAGS="$CFLAGS -O0"
elif test "$enable_debug" = "no"; then
	debug_status="no (no debug output, not testable, G_DISABLE_ASSERT)"
	AC_DEFINE_UNQUOTED(G_DISABLE_ASSERT, 1, [Disable glib assertions])
else
	debug_status="default (-g, debug output, testable)"
fi

AC_MSG_RESULT($debug_status)

AC_MSG_CHECKING(for more warnings)
if test "$enable_debug" != "no" -a "$GCC" = "yes"; then
	AC_MSG_RESULT(yes)
	CFLAGS="\
	-Wall \
	-Wchar-subscripts -Wmissing-declarations -Wmissing-prototypes \
	-Wnested-externs -Wpointer-arith \
	-Wcast-align -Wsign-compare \
	$CFLAGS"

	for option in -Wno-strict-aliasing -Wno-sign-compare; do
		SAVE_CFLAGS="$CFLAGS"
		CFLAGS="$CFLAGS $option"
		AC_MSG_CHECKING([whether gcc understands $option])
		AC_TRY_COMPILE([], [],
			has_option=yes,
			has_option=no,)
		if test $has_option = no; then
			CFLAGS="$SAVE_CFLAGS"
		fi
		AC_MSG_RESULT($has_option)
		unset has_option
		unset SAVE_CFLAGS
	done
	unset option
else
	AC_MSG_RESULT(no)
fi

AC_ARG_ENABLE(strict, [
               AS_HELP_STRING([--enable-strict], [Strict code compilation])
             ])

AC_MSG_CHECKING([build strict])

if test "$enable_strict" = "yes"; then
	CFLAGS="$CFLAGS -Werror \
		-DGTK_DISABLE_DEPRECATED \
		-DGDK_DISABLE_DEPRECATED \
		-DG_DISABLE_DEPRECATED \
		-DGDK_PIXBUF_DISABLE_DEPRECATED"
	TEST_MODE="thorough"
	INTROSPECTION_FLAGS="--warn-error"
	AC_DEFINE_UNQUOTED(WITH_STRICT, 1, [More strict checks])
	strict_status="yes (-Werror, thorough tests, fatals, no deprecations)"
else
	TEST_MODE="quick"
	INTROSPECTION_FLAGS=""
	strict_status="no (quick tests, non-fatal warnings)"
fi

AC_MSG_RESULT($strict_status)

AC_SUBST(INTROSPECTION_FLAGS)
AC_SUBST(TEST_MODE)

AC_MSG_CHECKING([whether to build with gcov testing])
AC_ARG_ENABLE([coverage],
		[AS_HELP_STRING([--enable-coverage], [Whether to enable coverage testing ])],
		[],
		[enable_coverage=no])

AC_MSG_RESULT([$enable_coverage])

AC_MSG_CHECKING([code coverage])

if test "$enable_coverage" = "yes"; then
	if test "$GCC" != "yes"; then
		AC_MSG_ERROR(Coverage testing requires GCC)
	fi

	AC_PATH_PROG(GCOV, gcov, no)
	if test "$GCOV" = "no" ; then
		AC_MSG_ERROR(gcov tool is not available)
	fi

	CFLAGS="$CFLAGS -O0 -g --coverage"
	LDFLAGS="$LDFLAGS --coverage"
fi

AM_CONDITIONAL([WITH_COVERAGE], [test "$enable_coverage" = "yes"])

AC_MSG_RESULT($enable_coverage)

# ----------------------------------------------------------------------
# Status

AC_SUBST(LIBRARY_CFLAGS)
AC_SUBST(LIBRARY_LIBS)

AC_OUTPUT([
Makefile
docs/Makefile
docs/reference/Makefile
docs/reference/gnome-keyring/Makefile
egg/Makefile
egg/tests/Makefile
library/Makefile
library/tests/Makefile
library/gnome-keyring-1.pc
library/gnome-keyring-1-uninstalled.pc
po/Makefile.in
])

# ------------------------------------------------------------------------------
# Summary
#

echo
echo "CFLAGS: $CFLAGS"
echo
echo "OPTIONS:"
echo "  Introspection:        $found_introspection"
echo "  Vala:                 $enable_vala"
echo "  Debug Build:          $debug_status"
echo "  Strict Build:         $strict_status"
echo "  Code Coverage:        $enable_coverage"
echo
