#                                               -*- Autoconf -*-
# Process this file with autoconf to produce a configure script.

AC_PREREQ([2.64])
AC_INIT([Virtual Smart Card], [0.7], [http://sourceforge.net/projects/vsmartcard/support], [virtualsmartcard], [http://vsmartcard.sourceforge.net/virtualsmartcard/README.html])
AC_CONFIG_SRCDIR([src/vpcd/ifd-vpcd.c])
AC_CONFIG_HEADERS([config.h])
AC_CONFIG_MACRO_DIR([m4])
AM_INIT_AUTOMAKE

dnl Add libtool support.
ifdef(
	[LT_INIT],
	[
		LT_INIT([win32-dll])
	],
	[
		AC_LIBTOOL_WIN32_DLL
		AC_PROG_LIBTOOL
	]
)

# Checks for programs.
AC_PROG_CC
AC_PROG_INSTALL
AC_PROG_LIBTOOL
AC_PROG_LN_S
AC_PROG_SED
AM_PATH_PYTHON
AC_PROG_CXX
AM_PROG_CC_C_O
PKG_PROG_PKG_CONFIG
AC_ARG_VAR([HELP2MAN],
           [absolute path to help2man used for man page generation of vicc])
AC_PATH_PROG(HELP2MAN, help2man, not found)
if test ! -r src/vpicc/vicc.1 -a "${HELP2MAN}" = "not found"
then
    AC_MSG_ERROR([Need help2man to generate man page for vicc])
fi

# If you need to see the details, just run make V=1.
m4_ifdef([AM_SILENT_RULES],[AM_SILENT_RULES([yes])])


# Checks for libraries.

# --enable-libpcsclite
AC_ARG_ENABLE(libpcsclite,
	AC_HELP_STRING([--enable-libpcsclite=DIR],[Build a lightweight version of
					libpcsclite, conflicts with PCSC-Lite @<:@default=no@:>@]),
	[libpcsclite="${enableval}"], [libpcsclite=no])
if test "${libpcsclite}" = no ; then
	PKG_CHECK_EXISTS([libpcsclite],
					 [PKG_CHECK_MODULES([PCSC], [libpcsclite])],
					 [AC_MSG_WARN([libpcsclite not found by pkg-config])])

	saved_CPPFLAGS="$CPPFLAGS"
	CPPFLAGS="$CPPFLAGS $PCSC_CFLAGS"
	AC_CHECK_HEADERS(ifdhandler.h,,
			[ AC_MSG_ERROR([ifdhandler.h not found, install libpcsclite or use ./configure PCSC_CFLAGS=...]) ])
	CPPFLAGS="$saved_CPPFLAGS"


	# --enable-serialdropdir=DIR
	AC_ARG_ENABLE(serialdropdir,
		AC_HELP_STRING([--enable-serialdropdir=DIR],[directory to install the
		serial wrapper driver (default to pcscd config or $(prefix)/pcsc/drivers/serial)]),
		[serialdropdir="${enableval}"], [serialdropdir=no])
	if test "${serialdropdir}" = no ; then
		if test "${prefix}" = NONE ; then
			serialdropdir="`$PKG_CONFIG libpcsclite --variable=usbdropdir`/serial"
			if test "${serialdropdir}" = "/serial" ; then
				AC_MSG_ERROR([use --enable-serialdropdir=DIR])
			fi
		else
			serialdropdir="${prefix}`$PKG_CONFIG libpcsclite --variable=usbdropdir`/serial"
			if test "${serialdropdir}" = "${prefix}/serial" ; then
				AC_MSG_ERROR([use --enable-serialdropdir=DIR])
			fi
		fi
	fi


	# --enable-serialconfdir=DIR
	AC_ARG_ENABLE(serialconfdir,
		AC_HELP_STRING([--enable-serialconfdir=DIR],[directory to install the
		serial wrapper driver (default to pcscd config or /etc/reader.conf.d)]),
		[serialconfdir="${enableval}"], [serialconfdir=no])
	if test "${serialconfdir}" = no ; then
		if test "${prefix}" = NONE ; then
			serialconfdir="`$PKG_CONFIG libpcsclite --variable=serialconfdir`"
			if test "${serialconfdir}" = "" ; then
				serialconfdir="/etc/reader.conf.d"
			fi
		else
			serialconfdir="${prefix}`$PKG_CONFIG libpcsclite --variable=serialconfdir`"
			if test "${serialconfdir}" = "${prefix}" ; then
				serialconfdir="${prefix}/etc/reader.conf.d"
			fi
		fi
	fi
else
    builddir="`pwd`"
	PCSC_CFLAGS="-DNO_LOG -I${builddir}/${srcdir}/src/libpcsclite/PCSC -I${builddir}/${srcdir}/src/libpcsclite"
	PCSC_LIBS="$builddir/src/libpcsclite/libpcsclite.la"
	AC_SUBST(PCSC_CFLAGS)
	AC_SUBST(PCSC_LIBS)
fi


# --enable-vpcdhost
AC_ARG_ENABLE(vpcdhost,
	AC_HELP_STRING([--enable-vpcdhost=ADRESS],[Default address to connect to when
					communicating with vicc. Use "/dev/null" if vpcd shall open
					the socket and wait for an incoming connection.
					@<:@default=/dev/null@:>@]),
	[vpcdhost="${enableval}"], [vpcdhost=/dev/null])
AC_SUBST(vpcdhost)
if test "${vpcdhost}" = "/dev/null"; then
	AC_DEFINE_UNQUOTED(VPCDHOST, NULL, [address of vicc])
else
	AC_DEFINE_UNQUOTED(VPCDHOST, "${vpcdhost}", [address of vicc])
fi


PACKAGE_SUMMARY="Smart card emulator written in Python"
AC_SUBST(PACKAGE_SUMMARY)


# Checks for header files.
AC_CHECK_HEADERS([arpa/inet.h stdint.h stdlib.h string.h sys/socket.h sys/time.h unistd.h])

# Checks for typedefs, structures, and compiler characteristics.
AC_TYPE_SIZE_T
AC_TYPE_UINT16_T
AC_CHECK_DECLS([SO_NOSIGPIPE], [], [], [#include <sys/socket.h>])
AC_CHECK_DECLS([MSG_NOSIGNAL], [], [], [#include <sys/socket.h>])
 
# Checks for library functions.
AC_FUNC_MALLOC
AC_CHECK_FUNCS([memset select socket])

# Select OS specific versions of source files.
AM_CONDITIONAL([BUILD_LIBPCSCLITE], [test "${libpcsclite}" = "yes"])
AC_SUBST(BUNDLE_HOST)
AC_SUBST(DYN_LIB_EXT)
AC_SUBST(LIB_PREFIX)
AC_SUBST(serialdropdir)
AC_SUBST(serialconfdir)
case "$host" in
*-*-darwin*)
	BUNDLE_HOST="MacOS"
	DYN_LIB_EXT="dylib"
	LIB_PREFIX="lib"
	;;
*-*-freebsd*)
	BUNDLE_HOST="FreeBSD"
	DYN_LIB_EXT="so"
	LIB_PREFIX="lib"
	;;
*-*-openbsd*)
	BUNDLE_HOST="OpenBSD"
	DYN_LIB_EXT="so.0.0"
	LIB_PREFIX="lib"
	;;
*-*-solaris*)
	BUNDLE_HOST="Solaris"
	DYN_LIB_EXT="so"
	LIB_PREFIX="lib"
	;;
*-*-dragonfly*)
	BUNDLE_HOST="DragonFly"
	DYN_LIB_EXT="so"
	LIB_PREFIX="lib"
	;;
*-*-cygwin*)
	BUNDLE_HOST="Cygwin"
	DYN_LIB_EXT="dll"
	LIB_PREFIX="cyg"
	;;
*)
	BUNDLE_HOST="Linux"
	DYN_LIB_EXT="so"
	LIB_PREFIX="lib"
esac

cat << EOF

${PACKAGE} has been configured with following options:

Version:              ${PACKAGE_VERSION}
User binaries:        $(eval eval eval echo "${bindir}")
PC/SC configuration:  ${serialconfdir}
Driver directory:     ${serialdropdir}
Python site-packages: $(eval eval eval echo "${pythondir}")

Build libpcsclite:    ${libpcsclite}
VPCD hostname: 	      ${vpcdhost}


Host:                 ${host}
Compiler:             ${CC}
Preprocessor flags:   ${CPPFLAGS}
Compiler flags:       ${CFLAGS}
Preprocessor flags:   ${CPPFLAGS}
Linker flags:         ${LDFLAGS}
Libraries:            ${LIBS}
PCSC_CFLAGS:          ${PCSC_CFLAGS}
PCSC_LIBS:            ${PCSC_LIBS}
BUNDLE_HOST:          ${BUNDLE_HOST}
LIB_PREFIX:           ${LIB_PREFIX}
DYN_LIB_EXT:          ${DYN_LIB_EXT}


HELP2MAN:             ${HELP2MAN}

EOF

AC_CONFIG_FILES([Makefile
                 doc/Makefile
                 doc/api/Makefile
                 npa-example-data/Makefile
                 npa-example-data/dh/Makefile
                 npa-example-data/ecdh/Makefile
                 src/Makefile
                 src/libpcsclite/Makefile
                 src/vpcd/Makefile
                 src/vpicc/Makefile
                 ])
AC_OUTPUT
