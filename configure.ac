#                                               -*- Autoconf -*-
# Process this file with autoconf to produce a configure script.

AC_PREREQ([2.63])
AC_INIT([sds-tests], [0.1], [],[],[]) 
AC_CONFIG_MACRO_DIR([m4])
LT_INIT

AC_CANONICAL_TARGET
AC_CANONICAL_SYSTEM
AC_CANONICAL_HOST

AM_INIT_AUTOMAKE([foreign subdir-objects -Wall])

# we should remove this soon, only needed for automake 1.10 and older
m4_ifdef([AM_SILENT_RULES], [AM_SILENT_RULES([yes])])

AC_CONFIG_SRCDIR([README.md])
AC_CONFIG_HEADERS([sds-tests-config.h])

# Checks for programs.
AC_PROG_CC
AM_PROG_CC_C_O
AC_PROG_CXX
AC_PROG_CXXCPP

AC_PROG_MKDIR_P
AC_REQUIRE_CPP

AC_CHECK_SIZEOF([long int])

AC_MSG_CHECKING([if nvcc is available])
AC_PATH_PROG([NVCC],[nvcc])
if test -z "$NVCC" ; then
        have_nvcc=0
        AC_MSG_RESULT(['nvcc' is not found in PATH])
else
        have_nvcc=1
        AC_MSG_RESULT(['nvcc' is found])
fi

dnl
dnl Verify pkg-config
dnl
PKG_PROG_PKG_CONFIG
if test "x$PKG_CONFIG" == "x"; then
   AC_MSG_ERROR([Could not find pkg-config utility!])
fi

PKG_CHECK_MODULES([MARGO],[margo],[have_margo=1],
   [AC_MSG_WARN([Could not find working margo installation])])
LIBS="$MARGO_LIBS $LIBS"
CPPFLAGS="$MARGO_CFLAGS $CPPFLAGS"
CFLAGS="$MARGO_CFLAGS $CFLAGS"

PKG_CHECK_MODULES([SSG],[ssg >= 0.5],[have_ssg=1],
   [AC_MSG_WARN([Could not find working ssg installation v0.5 or higher])])
LIBS="$SSG_LIBS $LIBS"
CPPFLAGS="$SSG_CFLAGS $CPPFLAGS"
CFLAGS="$SSG_CFLAGS $CFLAGS"

AC_CHECK_HEADERS(ssg-mpi.h,[have_ssg_mpi=1],
  [AC_MSG_WARN([Could not find ssg-mpi.h for MPI support in SSG])])

PKG_CHECK_MODULES([BAKESERVER],[bake-server],[have_bake_server=1],
        AC_MSG_WARN([Could not find working BAKE server installation]) )
LIBS="$BAKESERVER_LIBS $LIBS"
CPPFLAGS="$BAKESERVER_CFLAGS $CPPFLAGS"
CFLAGS="$BAKESERVER_CFLAGS $CFLAGS"

PKG_CHECK_MODULES([BAKECLIENT],[bake-client],[have_bake_client=1],
        AC_MSG_WARN([Could not find working BAKE client installation]) )
LIBS="$BAKECLIENT_LIBS $LIBS"
CPPFLAGS="$BAKECLIENT_CFLAGS $CPPFLAGS"
CFLAGS="$BAKECLIENT_CFLAGS $CFLAGS"

PKG_CHECK_MODULES([ARGOBOTS],[argobots],[have_argobots=1],
        AC_MSG_WARN([Could not find working Argobots installation]) )
LIBS="$ARGOBOTS_LIBS $LIBS"
CPPFLAGS="$ARGOBOTS_CFLAGS $CPPFLAGS"
CFLAGS="$ARGOBOTS_CFLAGS $CFLAGS"

# This is technically redundant with the above pkg-config check for
# Argobots, but it is a straightforward way to also get a config.h #define
# for the node-microbench utility to compile in optional Argobots tests
AC_CHECK_HEADERS(abt.h,,
  [AC_MSG_WARN([Could not find abt.h for Argobots])])

PKG_CHECK_MODULES([LIBPMEMOBJ],[libpmemobj],[have_libpmemobj=1],
   [AC_MSG_WARN([Could not find working libpmemobj installation])])
LIBS="$LIBPMEMOBJ_LIBS $LIBS"
CPPFLAGS="$LIBPMEMOBJ_CFLAGS $CPPFLAGS"
CFLAGS="$LIBPMEMOBJ_CFLAGS $CFLAGS"

AC_MSG_CHECKING([If MPI programs can be compiled])
AC_LINK_IFELSE(
    [AC_LANG_PROGRAM([[#include<mpi.h>]], [[MPI_Init(0,0);]])],
    [have_mpi=1
     AC_MSG_RESULT([yes])],
    [AC_MSG_WARN([Unable to compile MPI programs, try CC=mpicc])])

# Checks for header files.
AC_CHECK_HEADERS([x86intrin.h])

# Check support for -mrdseed
AS_IF([test "$ac_cv_header_x86intrin_h" = "yes"], [
  XXX_PROGRAM="
#ifdef HAVE_X86INTRIN_H
#include <x86intrin.h>
#endif
int main(void) {
    unsigned int flag;
    return __rdtscp(&flag);
}
"
  AC_MSG_CHECKING([if __rdtscp() intrinsic is supported])
  AC_COMPILE_IFELSE(
    [AC_LANG_SOURCE([$XXX_PROGRAM])],
    AC_MSG_RESULT([yes])
    AC_DEFINE(HAVE_RDTSCP_INTRINSIC, 1, Define if __rdtscp() intrinsic is available),
    [AC_MSG_RESULT([no])]
  )
])

# enable or disable test programs that depend on different combinations of
# functionality
AM_CONDITIONAL([HAVE_MARGO], [test "x${have_mpi}" = x1 && test "x${have_margo}" = x1 && test "x${have_ssg}" = x1 && test "x${have_ssg_mpi}" = x1])
AM_CONDITIONAL([HAVE_BAKE], [test "x${have_mpi}" = x1 && test "x${have_bake_client}" = x1 && test "x${have_bake_server}" = x1 && test "x${have_ssg}" = x1 && test "x${have_ssg_mpi}" = x1])
AM_CONDITIONAL([HAVE_PMDK], [test "x${have_libpmemobj}" = x1 && test "x${have_argobots}" = x1])
AM_CONDITIONAL([HAVE_SSG], [test "x${have_ssg}" = x1 && test "x${have_ssg_mpi}" = x1])
AM_CONDITIONAL([HAVE_MPI], [test "x${have_mpi}" = x1])
AM_CONDITIONAL([HAVE_GPU_TESTING], [test "x${have_nvcc}" = x1 && test "x${have_mpi}" = x1])



AC_CONFIG_FILES([Makefile])
AC_OUTPUT
