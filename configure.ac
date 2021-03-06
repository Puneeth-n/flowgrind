AC_INIT([Flowgrind], [0.7.1], [https://github.com/flowgrind/flowgrind/issues],
        [flowgrind], [https://www.flowgrind.net])
AC_CONFIG_AUX_DIR([build-aux])
AM_INIT_AUTOMAKE([foreign -Wall -Werror no-dist])

# Minimum Autoconf version required
AC_PREREQ([2.61])

# Files to generate
AC_CONFIG_HEADERS([config.h])
AC_CONFIG_FILES([ \
    Makefile \
    doc/Makefile \
    src/Makefile \
])

# Checking for programs
AC_PROG_CC
AM_PROG_CC_C_O
AC_PROG_CPP
AC_USE_SYSTEM_EXTENSIONS
AC_PROG_MAKE_SET
AC_PATH_TOOL([PERL], [perl])

# Checking for optional use of doxygen
AC_ARG_WITH([doxygen],
            [AS_HELP_STRING([--without-doxygen],
                            [ignore presence of doxygen and disable doxygen \
                             feature to generate API documentation])])

AS_IF([test "x$with_doxygen" != "xno"],
      [AC_PATH_TOOL([DOXYGEN], [doxygen])],
      [DOXYGEN=""])

AS_IF([test -n "$DOXYGEN"],
      [AC_DEFINE([HAVE_DOXYGEN], [1], [Define to 1 if you have doxygen installed])],
      [AS_IF([test "x$with_doxygen" = "xyes"],
             [AC_MSG_ERROR([doxygen requested but not found])])
      ])
AM_CONDITIONAL([USE_DOXYGEN], [test "x$with-doxygen" != "xno" -a -n "$DOXYGEN"])

# Checking for host-system types, includes, and libs
AC_CANONICAL_HOST
case $host in
    *darwin*)
        AC_DEFINE([__DARWIN__], [1], [Darwin])
        AC_DEFINE([_BSD_SOCKLEN_T_], [], [Need this under Darwin so that \
                                          socklen_t will be defined in <sys/socket.h>])
        # if Macport is used
        AS_IF([test -d /opt/local/lib],
              [AC_SUBST([LDFLAGS], [-L/opt/local/lib])])
        AS_IF([test -d /opt/local/include],
              [AC_SUBST([CFLAGS], [-I/opt/local/include])
               AC_SUBST([CPPFLAGS], [-I/opt/local/include])])
        ;;

    *freebsd*)
        AC_DEFINE([__FREEBSD__], [1], [FreeBSD])
        AC_SUBST([LDFLAGS], [-L/usr/local/lib])
        AC_SUBST([CFLAGS], [-I/usr/local/include])
        AC_SUBST([CPPFLAGS], [-I/usr/local/include])
        ;;

    *linux*)
        AC_DEFINE([__LINUX__], [1], [Linux])
        ;;
esac

# Checking for C features
AC_C_CONST
AC_C_INLINE

# Checking for GNU extensions
AC_GNU_SOURCE

# Checking functions
AC_FUNC_ERROR_AT_LINE
AC_FUNC_FORK
AC_FUNC_MALLOC
AC_FUNC_MEMCMP
AC_FUNC_SELECT_ARGTYPES
AC_FUNC_STRFTIME
AC_CHECK_FUNCS([ \
    bzero \
    memset \
    pselect \
    socket \
    strchr \
    strerror \
    uname \
    asprintf \
    strdup \
], [], [AC_MSG_ERROR([required function not found])])

# Checking for structs
AC_STRUCT_TM

# Checking for types
AC_TYPE_SIGNAL
AC_TYPE_SIZE_T
AC_TYPE_SSIZE_T
AC_TYPE_INT8_T
AC_TYPE_UINT32_T
AC_TYPE_INT32_T
AC_TYPE_UNSIGNED_LONG_LONG_INT

# Checking fot headers
AC_HEADER_SYS_WAIT
AC_HEADER_TIME
AC_HEADER_STDC
AC_CHECK_HEADERS([ \
    arpa/inet.h \
    errno.h \
    fcntl.h \
    fenv.h \
    float.h \
    limits.h \
    math.h \
    netdb.h \
    netinet/in.h \
    netinet/in_systm.h \
    netinet/tcp.h \
    pthread.h \
    signal.h \
    stdarg.h \
    stdio.h \
    stdlib.h \
    string.h \
    sys/ioctl.h \
    syslog.h \
    sys/param.h \
    sys/time.h \
    sys/uio.h \
    sys/utsname.h \
    time.h \
], [], [AC_MSG_ERROR([required header not found])])

AC_CHECK_HEADERS([net/if.h], [],
                 [AC_MSG_ERROR([required header not found])], [
    #include <stdio.h>
    #include <stdlib.h>
    #include <stddef.h>
    #include <sys/types.h>
    #include <sys/socket.h>
])

AC_CHECK_HEADERS([netinet/if_ether.h netinet/ip.h], [],
                 [AC_MSG_ERROR([required header not found])], [
    #include <stdio.h>
    #include <stdlib.h>
    #include <stddef.h>
    #include <sys/types.h>
    #include <sys/socket.h>
    #include <netinet/in.h>
    #include <netinet/in_systm.h>
])

# Checking for libraries
AC_SEARCH_LIBS([log], [m])
AC_SEARCH_LIBS([clock_gettime], [rt])
AC_SEARCH_LIBS([socket], [socket])
AC_SEARCH_LIBS([pthread_create], [pthread])

AC_SUBST([LIBS])

# Checking for feature --enable-debug
AC_ARG_ENABLE([debug], [AS_HELP_STRING([--enable-debug],
                                       [enable debugging output and symbols])])

AS_IF([test "x$enable_debug" = "xyes"],
      [AC_DEFINE([DEBUG], [1], [enable debugging output])])

# Checking for libpcap
AC_ARG_WITH([pcap],
            [AS_HELP_STRING([--without-pcap],
                            [ignore presence of libpcap and disable packet \
                             capturing feature])])

AS_IF([test "x$with_pcap" != "xno"],
      [AC_CHECK_HEADER([pcap/pcap.h],
                       [AC_CHECK_LIB([pcap], [pcap_create], [have_pcap=yes],
                                     [have_pcap=no;
                                      AC_MSG_WARN([could not find libpcap: will compile without\
                                                   optional traffic dump feature])],
                                     [-lpcap])
                       ],
                       [have_pcap=no;
                        AC_MSG_WARN([header file pcap.h not found: will compile \
                                     without optional traffic dump feature])
                       ])
      ],
      [have_pcap=no])

AS_IF([test "x$have_pcap" = "xyes"],
      [AC_DEFINE([HAVE_LIBPCAP], [1],
                 [Define to 1 if you have libpcap installed (-lpcap)])

       AS_IF([test -z "$PCAP_CONFIG"],
             [AC_PATH_TOOL([PCAP_CONFIG], [pcap-config], [no])])

       AS_IF([test "x$PCAP_CONFIG" = "xno"],
             [AC_MSG_NOTICE([pcap-config not found in path])],
             [PCAP_CFLAGS=`$PCAP_CONFIG --cflags`
              PCAP_LDADD=`$PCAP_CONFIG --libs`

              AC_SUBST([PCAP_CFLAGS])
              AC_SUBST([PCAP_LDADD])
             ])
      ],
      [AS_IF([test "x$with_pcap" = "xyes"],
             [AC_MSG_ERROR([libpcap requested but not found])])
      ])
AM_CONDITIONAL([USE_LIBPCAP], [test "x$with_pcap" != "xno" -a "x$have_pcap" = "xyes"])

# Checking for gsl
AC_ARG_WITH([gsl],
            [AS_HELP_STRING([--without-gsl],
                            [ignore presence of libgsl and disable usage of \
                             GNU Scientific Library])])

AS_IF([test "x$with_gsl" != "xno"],
      [AC_CHECK_HEADER([gsl/gsl_rng.h],
                       [AC_CHECK_LIB([gsl], [gsl_rng_alloc], [have_gsl=yes],
                                     [have_gsl=no;
                                      AC_MSG_WARN([could not find libgsl: will compile without\
                                                   optional traffic trafgen distributions])],
                                     [-lgslcblas])
                       ],
                       [have_gsl=no;
                        AC_MSG_WARN([header file gsl_rng.h not found: will compile \
                                     without optional traffic trafgen distributions])
                       ])
      ],
      [have_gsl=no])

AS_IF([test "x$have_gsl" = "xyes"],
      [AC_DEFINE([HAVE_LIBGSL], [1],
                 [Define to 1 if you have libgsl installed (-lgsl)])

       AS_IF([test -z "$GSL_CONFIG"],
             [AC_PATH_TOOL([GSL_CONFIG], [gsl-config], [no])])

       AS_IF([test "x$GSL_CONFIG" = "xno"],
             [AC_MSG_NOTICE([gsl-config not found in path])],
             [GSL_CFLAGS=`$GSL_CONFIG --cflags`
              GSL_LDADD=`$GSL_CONFIG --libs`

              AC_SUBST([GSL_CFLAGS])
              AC_SUBST([GSL_LDADD])
             ])
      ],
      [AS_IF([test "x$with_gsl" = "xyes"],
             [AC_MSG_ERROR([libgsl requested but not found])])
      ])
AM_CONDITIONAL([USE_LIBGSL], [test "x$with_gsl" != "xno" -a "x$have_gsl" = "xyes"])

# Checking for CURL
AS_IF([test -z "$CURL_CONFIG"],
      [AC_PATH_TOOL([CURL_CONFIG], [curl-config], [no])])

AS_IF([test "x$CURL_CONFIG" = "xno"],
      [AC_MSG_ERROR([required curl-config not found in path])])

CURL_CFLAGS=`$CURL_CONFIG --cflags`
CURL_LDADD=`$CURL_CONFIG --libs`

AC_SUBST([CURL_CFLAGS])
AC_SUBST([CURL_LDADD])

# Checking for xmlrpc-c
AS_IF([test -z "$XMLRPC_C_CONFIG"],
      [AC_PATH_TOOL([XMLRPC_C_CONFIG], [xmlrpc-c-config], [no])])

AS_IF([test "x$XMLRPC_C_CONFIG" = "xno"],
      [AC_MSG_ERROR([required xmlrpc-c-config not found in path])])

AS_IF([! $XMLRPC_C_CONFIG --features | grep "curl-client" > /dev/null],
      [AC_MSG_ERROR([xmlrpc-c needs to be compiled with curl-client enabled])])

AS_IF([! $XMLRPC_C_CONFIG --features | grep "abyss-server" > /dev/null],
      [AC_MSG_ERROR([xmlrpc-c needs to be compiled with abyss-server enabled])])

XMLRPC_C_VERSION=`$XMLRPC_C_CONFIG --version`
AC_MSG_NOTICE([xmlrpc-c found in version $XMLRPC_C_VERSION])
AC_DEFINE_UNQUOTED([XMLRPC_C_VERSION], [$XMLRPC_C_VERSION], [the current xmlrpc-c version])

AC_CHECK_MEMBERS([struct xmlrpc_curl_xportparms.dont_advertise], [], [], [
    #include <xmlrpc-c/client.h>
    #include <xmlrpc-c/transport.h>
])

XMLRPC_C_CLIENT_CFLAGS=`$XMLRPC_C_CONFIG client --cflags`
XMLRPC_C_CLIENT_LDADD=`$XMLRPC_C_CONFIG client --ldadd`
XMLRPC_C_SERVER_CFLAGS=`$XMLRPC_C_CONFIG abyss-server --cflags`
XMLRPC_C_SERVER_LDADD=`$XMLRPC_C_CONFIG abyss-server --ldadd`

AC_SUBST([XMLRPC_C_CLIENT_CFLAGS])
AC_SUBST([XMLRPC_C_CLIENT_LDADD])
AC_SUBST([XMLRPC_C_SERVER_CFLAGS])
AC_SUBST([XMLRPC_C_SERVER_LDADD])

AC_OUTPUT
