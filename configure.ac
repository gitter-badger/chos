****************************************************************************/
# Initialization
#****************************************************************************/
m4_define(PROJECT_NAME,  [CHOS])
m4_define(PROJECT_EMAIL, [chos@nersc.gov])
m4_define(PROJECT_TAR,   [chos])
m4_define(PROJECT_MAJOR, 0)
m4_define(PROJECT_MINOR, 13)
m4_define(PROJECT_MICRO, 2)

AC_INIT(PROJECT_NAME, PROJECT_MAJOR.PROJECT_MINOR.PROJECT_MICRO,
        PROJECT_EMAIL, PROJECT_TAR)

AC_CONFIG_AUX_DIR(config)
AC_CONFIG_MACRO_DIR([m4])

AM_INIT_AUTOMAKE([foreign])

AM_CONFIG_HEADER(config.h)


#****************************************************************************/
# OS Check
#****************************************************************************/
AC_MSG_CHECKING([OS])
os=`uname -s`
AC_MSG_RESULT([$os])
if test ! "$os" = "Linux"
then
    AC_MSG_ERROR([this application will only work on Linux!])
fi

MKDIR="mkdir -p"                ; AC_SUBST(MKDIR)
SHLIBMODE=755                      ; AC_SUBST(SHLIBMODE)

AC_PROG_AWK
AC_PROG_CC
AC_HEADER_STDC

AC_PROG_LIBTOOL

GCC_WARNINGS="-Wall -Wwrite-strings -Werror \
        -Wpointer-arith -Wcast-qual -Wcast-align \
        -Wstrict-prototypes -Wmissing-prototypes \
        -Wnested-externs -Winline -Wshadow"

if test "x$GCC" = "xyes"; then
	CC=gcc                          ; AC_SUBST(CC)
	OS_CFLAGS=
	LD_D="$CC -shared -Xlinker -x"
	WARNINGS="$GCC_WARNINGS"
	PIC="-fPIC"
	DYNTYPE=so
	LD=ld
	LD_L="$CC -shared $LDFLAGS"
	RANLIB=:
	STRIP=strip
	CC_STATIC="-Xlinker -export-dynamic"
else
  AC_MSG_ERROR([Things probably won't work with a non-gcc compiler!])
fi

AC_SUBST(DYNTYPE)
AC_SUBST(OS_CFLAGS)
AC_SUBST(WARNINGS)
AC_SUBST(PIC)
AC_SUBST(LD)
AC_SUBST(LD_D)
AC_SUBST(LD_L)
AC_SUBST(RANLIB)
AC_SUBST(STRIP)
AC_SUBST(CC_STATIC)


dnl Check for libraries

dnl Optional stuff


#****************************************************************************/
# Module directory
#****************************************************************************/
AC_MSG_CHECKING([module directory])
AC_ARG_WITH(
    [moduledir],
    [  --with-moduledir=DIR specify where to install the kernel module],
    [MODULES_DIR=$withval],
    [MODULES_DIR=/lib/modules/$KERNEL_VERSION/kernel/misc])
AC_MSG_RESULT([$MODULES_DIR])
AC_SUBST(MODULES_DIR)

#****************************************************************************/
# Build kernel module
#****************************************************************************/
AC_ARG_ENABLE(kernel,
     [  --enable-kernel    Build kernel module],
     [case "${enableval}" in
       yes) BUILD_KERNEL=1
            #****************************************************************************/
            # Kernel Source Check
            #****************************************************************************/
            AC_MSG_CHECKING([kernel source])
            AC_ARG_WITH(
                [kernel-source],
            [  --with-kernel-source[=PATH]  directory where kernel source exists],
                [KERNEL_BASE=$withval],
                [KERNEL_BASE=/lib/modules/`uname -r`/build])
            KERNEL_HDR="$KERNEL_BASE/include"
            if test -f "$KERNEL_HDR/linux/utsrelease.h"
            then
                KERNEL_VERSION_H="$KERNEL_HDR/linux/utsrelease.h"
            elif test -f "$KERNEL_HDR/generated/utsrelease.h"
            then
                KERNEL_VERSION_H="$KERNEL_HDR/generated/utsrelease.h"
            else
                AC_MSG_RESULT([not found])
                AC_MSG_ERROR([cannot find the kernel headers in $KERNEL_HDR. Use --with-kernel-source])
            fi

            AC_MSG_RESULT([found in $KERNEL_HDR ($KERNEL_VERSION_H)])

            KERNEL_VERSION=`$CC -E -dM -I$KERNEL_HDR $KERNEL_VERSION_H|$AWK '/UTS_RELEASE/ { print $3 }' \
                   | sed 's/"//g'`
            AC_MSG_RESULT(kernel version: $KERNEL_VERSION)
            AC_SUBST(KERNEL_BASE)
            AC_SUBST(KERNEL_HDR)
            AC_SUBST(KERNEL_VERSION)
            ;;
       no)  BUILD_KERNEL=0 ;;
       *) AC_MSG_ERROR(bad value ${enableval} for --enable-kernel) ;;
     esac],[BUILD_KERNEL=0])
AC_SUBST(BUILD_KERNEL)

kernel_module_enabled=$(test $BUILD_KERNEL -gt 0 && echo yes || echo no)

dnl packaging convenience
AC_ARG_ENABLE(fakeroot,
[  --enable-fakeroot=<path to packaging directory>], FAKEROOT=$enableval)
AC_SUBST(FAKEROOT)

#****************************************************************************/
# PAM integration
#****************************************************************************/
AC_CHECK_LIB([pam], [pam_authenticate], PAM_LIBS="-lpam",
             AC_MSG_FAILURE([Couldn't find -lpam]))

AC_CHECK_LIB([pam_misc], [pam_misc_setenv], PAM_MISC_LIBS="-lpam_misc",
             AC_MSG_FAILURE([Couldn't find -lpam_misc]))

AC_SUBST(PAM_LIBS)
AC_SUBST(PAM_MISC_LIBS)

pam_enabled="yes"


AC_ARG_ENABLE(securedir,
[  --enable-securedir=<path to location of PAMs> [default \$libdir/security]],
        SECUREDIR=$enableval, SECUREDIR=$libdir/security)
AC_SUBST(SECUREDIR)


AC_OUTPUT(Makefile utils/Makefile Make.Rules conf/Makefile pam_chos/Makefile kernel/dkms.conf)

AC_MSG_RESULT([

*********************************************
* PROJECT_NAME $VERSION configuration:
*********************************************
*   Enable kernel module.................$kernel_module_enabled
*   Enable PAM support...................$pam_enabled
*********************************************


])
