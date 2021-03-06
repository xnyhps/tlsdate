AC_INIT([tlsdate],[0.0.6],[jacob at appelbaum.net])
AC_CONFIG_AUX_DIR([config])
AC_CONFIG_MACRO_DIR([m4])

AC_CANONICAL_TARGET
AC_ARG_PROGRAM
AC_USE_SYSTEM_EXTENSIONS

AM_INIT_AUTOMAKE([-Wall -Werror -Wno-portability subdir-objects foreign tar-ustar])

AC_PREREQ([2.63])

AC_CONFIG_HEADERS([config.h:config.in])dnl Keep filename to 8.3 for MS-DOS.

PKG_PROG_PKG_CONFIG
LT_PREREQ([2.2])
LT_INIT
LT_LANG([C])
gl_VISIBILITY
m4_ifdef([AM_SILENT_RULES], [AM_SILENT_RULES([yes])])

CONFIG_EXTRA
AX_PLATFORM

dnl Here we should build a small program to fetch the build system time in a portable
dnl manner. We have no Win32 users, we can fix this if we ever find one that
dnl cares.
COMPILE_DATE=`date +%s`
AC_SUBST([COMPILE_DATE])
AC_DEFINE_UNQUOTED([RECENT_COMPILE_DATE],
                   [(uint32_t) ${COMPILE_DATE}],
                   [Time in seconds since the Disco epoch at build time])

dnl Build up the directory we will use to install certs
TLSDATE_CA_ROOTS="${sysconfdir}/$PACKAGE_NAME/ca-roots"
AC_SUBST([TLSDATE_CA_ROOTS])

dnl Place we install our config file
TLSDATE_CONF_DIR="${sysconfdir}/$PACKAGE_NAME/"
AC_SUBST([TLSDATE_CONF_DIR])

dnl check for PolarSSL
OPT_POLARSSL=yes

AC_MSG_CHECKING([PolarSSL])
AC_ARG_WITH([polarssl],
            [AS_HELP_STRING([--with-polarssl=DIR],
                            [where to look for PolarSSL, DIR points to the installation root])])

AS_CASE([$with_polarssl],
        [""|yes|no], [POLARSSL_DIR=""],
        [*], [POLARSSL_DIR=$with_polarssl])
OPT_POLARSSL=$with_polarssl

SSL_FLAGS=""
SSL_LDFLAGS=""
SSL_LIBS="-lssl -lcrypto"

AS_IF([test "x${OPT_POLARSSL}" != "xno"], [
  AS_IF([test -z "${POLARSSL_DIR}"], [
    dnl check for lib first without setting any new path
    AC_CHECK_LIB(polarssl, ssl_init,
    dnl libpolarssl found, set the variable
     [
       AC_DEFINE(USE_POLARSSL, 1, [if PolarSSL is enabled])
       AC_SUBST(USE_POLARSSL, [1])
       POLARSSL_ENABLED=1
       USE_POLARSSL="yes"
     ])
  ])

  addld=""
  addlib=""
  addcflags=""
  polarssllib=""

  AS_IF([test "x${USE_POLARSSL}" != "xyes"], [
    dnl add the path and test again
    addld=-L${POLARSSL_DIR}/lib$libsuff
    addcflags=-I${POLARSSL_DIR}/include
    polarssllib=${POLARSSL_DIR}/lib$libsuff

    LDFLAGS="$LDFLAGS $addld"
    AS_IF([test "$addcflags" != "-I/usr/include"], [
        AX_APPEND_COMPILE_FLAGS(["$addcflags"])
       CPPFLAGS="$CPPFLAGS $addcflags"
    ])

    AC_CHECK_LIB(polarssl, ssl_init,
     [
       AC_DEFINE(USE_POLARSSL, 1, [if PolarSSL is enabled])
       AC_SUBST(USE_POLARSSL, [1])
       POLARSSL_ENABLED=1
       USE_POLARSSL="yes"
       SSL_CFLAGS=$addcflags
       SSL_LDFLAGS=$addld
     ])
  ])

  AS_IF([test "x${USE_POLARSSL}" = "xyes"], [
    AC_MSG_NOTICE([detected PolarSSL])
    SSL_LIBS="-lpolarssl"

    AS_IF([test -n "$polarssllib"], [
      dnl when shared libs were found in a path that the run-time
      dnl linker doesn't search through, we need to add it to
      dnl LD_LIBRARY_PATH to prevent further configure tests to fail
      dnl due to this
      AS_IF([test "x$cross_compiling" != "xyes"], [
        LD_LIBRARY_PATH="$LD_LIBRARY_PATH:$polarssllib"
        export LD_LIBRARY_PATH
        AC_MSG_NOTICE([Added $polarssllib to LD_LIBRARY_PATH])
      ])
    ])
  ])
])
AC_SUBST(SSL_CFLAGS)
AC_SUBST(SSL_LDFLAGS)
AC_SUBST(SSL_LIBS)
AM_CONDITIONAL(POLARSSL, test "x${USE_POLARSSL}" = "xyes")

dnl Required headers
AS_IF([test "x${USE_POLARSSL}" != "xyes"], [
  dnl First check to see if openssl is installed
  AC_CHECK_HEADERS([openssl/ssl.h], ,[AC_MSG_ERROR([OpenSSL is not installed, openssl/sslh is missing])])
  AC_CHECK_HEADERS([openssl/bio.h], ,[AC_MSG_ERROR([Required headers missing; compilation will not succeed])])
  AC_CHECK_HEADERS([openssl/err.h], ,[AC_MSG_ERROR([Required headers missing; compilation will not succeed])])
  AC_CHECK_HEADERS([openssl/evp.h], ,[AC_MSG_ERROR([Required headers missing; compilation will not succeed])])
])

AC_CHECK_HEADERS([arpa/inet.h], ,[AC_MSG_ERROR([Required headers missing; compilation will not succeed])])
AC_CHECK_HEADERS([getopt.h], ,[AC_MSG_ERROR([Required headers missing; compilation will not succeed])])
AC_CHECK_HEADERS([grp.h], ,[AC_MSG_ERROR([Required headers missing; compilation will not succeed])])
AC_CHECK_HEADERS([pwd.h], ,[AC_MSG_ERROR([Required headers missing; compilation will not succeed])])
AC_CHECK_HEADERS([stdint.h], ,[AC_MSG_ERROR([Required headers missing; compilation will not succeed])])
AC_CHECK_HEADERS([stdio.h], ,[AC_MSG_ERROR([Required headers missing; compilation will not succeed])])
AC_CHECK_HEADERS([stdlib.h], ,[AC_MSG_ERROR([Required headers missing; compilation will not succeed])])
AC_CHECK_HEADERS([sys/mman.h], ,[AC_MSG_ERROR([Required headers missing; compilation will not succeed])])
AC_CHECK_HEADERS([sys/time.h], ,[AC_MSG_ERROR([Required headers missing; compilation will not succeed])])
AC_CHECK_HEADERS([sys/types.h], ,[AC_MSG_ERROR([Required headers missing; compilation will not succeed])])
AC_CHECK_HEADERS([sys/wait.h], ,[AC_MSG_ERROR([Required headers missing; compilation will not succeed])])
AC_CHECK_HEADERS([time.h], ,[AC_MSG_ERROR([Required headers missing; compilation will not succeed])])
AC_CHECK_HEADERS([unistd.h], ,[AC_MSG_ERROR([Required headers missing; compilation will not succeed])])

AC_CHECK_FUNCS([setresuid])
AC_CHECK_FUNCS([gettimeofday])


case "$host" in
  *-darwin*)
    dnl This is for Mac OS X (10.8.2)
    AC_CHECK_HEADERS([mach/clock.h], ,[AC_MSG_ERROR([Required headers missing; compilation will not succeed])])
    AC_CHECK_HEADERS([mach/mach.h], ,[AC_MSG_ERROR([Required headers missing; compilation will not succeed])])
    AC_ARG_WITH([unpriv-group],
                [AS_HELP_STRING([--with-unpriv-group=<group>],
                                [Group to drop privs to @<:@default: nogroup@:>@])])
    AS_CASE([$with_unpriv_group],
            [""|yes|no], [UNPRIV_GROUP="nobody"],
            [*], [UNPRIV_GROUP=$with_unpriv_group])
    AC_DEFINE_UNQUOTED([UNPRIV_GROUP], ["${UNPRIV_GROUP}"], [Unprivileged group])
    ;;
  *-linux*)
    dnl This is for GNU/Linux
    dnl Check for clock_gettime.  Some systems put it into -lc, while
    dnl others use -lrt.  Try the first and fallback to the latter.
    RT_LIB=
    AC_CHECK_FUNC([clock_gettime], [:],
                  [AC_CHECK_LIB([rt], [clock_gettime], [RT_LIB="-lrt"],
                  [AC_MSG_ERROR([Your system lacks clock_gettime])])])
    AC_SUBST(RT_LIB)
    AC_ARG_WITH([unpriv-group],
                [AS_HELP_STRING([--with-unpriv-group=<group>],
                [Group to drop privs to @<:@default: nogroup@:>@])])
    AS_CASE([$with_unpriv_group],
            [""|yes|no], [UNPRIV_GROUP="nogroup"],
            [*], [UNPRIV_GROUP=$with_unpriv_group])
    AC_DEFINE_UNQUOTED([UNPRIV_GROUP], ["${UNPRIV_GROUP}"], [Unprivileged group])
    ;;
esac

AC_MSG_CHECKING([user/group to drop privs to])

AC_ARG_WITH([unpriv-user],
            [AS_HELP_STRING([--with-unpriv-user=<user>],
                            [User to drop privs to @<:@default: nobody@:>@])])
AS_CASE([$with_unpriv_user],
        [""|yes|no], [UNPRIV_USER="nobody"],
        [*], [UNPRIV_USER=$with_unpriv_user])
AC_DEFINE_UNQUOTED([UNPRIV_USER], ["${UNPRIV_USER}"], [Unprivileged user])
AC_MSG_RESULT(${UNPRIV_USER}:${UNPRIV_GROUP})

AC_ARG_ENABLE([dbus],
	      [AS_HELP_STRING([--disable-dbus],
			      [Disable automatically dbus support])])
AS_IF([test "x$enable_dbus" = xyes], [
    PKG_CHECK_MODULES([DBUS], [dbus-1], [
	    AC_DEFINE([HAVE_DBUS], [1], [Enable dbus support])
	    AC_MSG_CHECKING([user/group to use for dbus])
	    AC_ARG_WITH([dbus-user],
			[AS_HELP_STRING([--with-dbus-user=<user>],
					[User to send dbus signals from @<:@default: nobody@:>@])])
	    AS_CASE([$with_dbus_user],
		    [""|yes|no], [DBUS_USER="nobody"],
		    [*], [DBUS_USER=$with_dbus_user])
	    AC_ARG_WITH([dbus-group],
			[AS_HELP_STRING([--with-dbus-group=<group>],
					[Group to send dbus signals from @<:@default: nogroup@:>@])])
	    AS_CASE([$with_dbus_group],
		    [""|yes|no], [DBUS_GROUP="nogroup"],
		    [*], [DBUS_GROUP=$with_dbus_group])
	    AC_MSG_RESULT(${DBUS_USER}:${DBUS_GROUP})
	    AC_DEFINE_UNQUOTED([DBUS_USER], ["${DBUS_USER}"], [DBus user])
	    AC_DEFINE_UNQUOTED([DBUS_GROUP], ["${DBUS_GROUP}"], [DBus group])
	], [
	    AS_IF([test "x$enable_dbus" = xyes],
		  [AC_MSG_ERROR([dbus requested but not found])])
	])
    ])
AC_SUBST(DBUS_CFLAGS)
AC_SUBST(DBUS_LIBS)

dnl Debug and hardening flags all in one shot
dnl Always do this at the end, otherwise you end up filtering system/other libraries
AC_ARG_ENABLE([hardened-checks],
              [AS_HELP_STRING([--disable-hardened-checks],
                              [Disable automatically enabling hardened toolchain options])])
AC_DEFUN([LOCAL_CHECK_FLAGS],[
          AC_REQUIRE([AX_CHECK_LINK_FLAG])
          AC_REQUIRE([AX_APPEND_COMPILE_FLAGS])
          AC_LANG_PUSH([C])
          AS_IF([test "x$enable_hardened_checks" != xno], [
              CFLAGS=
              LIBS=
              AX_APPEND_COMPILE_FLAGS([-g -O1])
          ], [
              AC_MSG_WARN([using hardened flags is HIGHLY RECOMMENDED and disabling them is a BAD IDEA])
          ])
          AX_APPEND_COMPILE_FLAGS([-Wall -fno-strict-aliasing])
          AS_IF([test "x$enable_hardened_checks" != xno], [
              AX_APPEND_COMPILE_FLAGS([-D_FORTIFY_SOURCE=2 -fstack-protector-all])
              AX_APPEND_COMPILE_FLAGS([-fwrapv -fPIE -Wstack-protector])
              AX_APPEND_COMPILE_FLAGS([--param=ssp-buffer-size=1])
              AX_CHECK_LINK_FLAG([-z relro -z now])
              AX_CHECK_LINK_FLAG([-pie])
          ])
          AC_LANG_POP
          ])
LOCAL_CHECK_FLAGS

AC_ARG_ENABLE([code-coverage-checks],
              [AS_HELP_STRING([--enable-code-coverage-checks],
                              [Enable gcov/lcov compile time options])],
              [AX_APPEND_COMPILE_FLAGS([-ftest-coverage -fprofile-arcs])])

AC_CONFIG_FILES([Makefile])
AC_OUTPUT
