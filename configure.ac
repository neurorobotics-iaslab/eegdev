# - If the library source code has changed at all since the last update,
# then increment revision.
# - If any interfaces have been added, removed, or changed since the last
# update, increment current, and set revision to 0.
# - If any interfaces have been added since the last public release, then
# increment age.
# - If any interfaces have been removed since the last public release, then
# set age to 0.

m4_define([lib_current],0)
m4_define([lib_revision],3)
m4_define([lib_age],0)

# Setup autoconf
AC_INIT(eegdev,[lib_current.lib_revision],[nicolas.bourdaud@epfl.ch])
AC_CONFIG_SRCDIR([src/core/eegdev.h])
AC_CONFIG_LIBOBJ_DIR([lib])
AC_CONFIG_MACRO_DIR([m4])
AC_CONFIG_AUX_DIR([build-aux])
AC_CONFIG_HEADERS([config/config.h])

# Setup automake
AM_INIT_AUTOMAKE([no-dist-gzip dist-bzip2 color-tests subdir-objects])
m4_ifdef([AM_SILENT_RULES], [AM_SILENT_RULES([yes])])

AC_CANONICAL_HOST
AC_CANONICAL_BUILD

# Check for programs
AC_PROG_CC_C99
gl_EARLY
LT_INIT([win32-dll disable-static])
LT_SYS_MODULE_EXT
AC_SUBST(LIBTOOL_DEPS)
AC_CHECK_PROG(HAS_FLEX,flex,yes,no)
AC_CHECK_PROG(HAS_BISON,bison,yes,no)
gl_INIT

#AC_C_BIGENDIAN
AC_DEF_API_EXPORT_ATTRS

# Test for the correct way to set CLOEXEC to file descriptors
AC_CHECK_DECLS([O_CLOEXEC, FD_CLOEXEC], [], [], [[#include <fcntl.h>]])


# pthread is needed in core and modules
AC_SEARCH_LIBS([pthread_create], [pthread posix4], [],
               AC_MSG_ERROR([The pthread library has not been found]))

AC_CHECK_FUNC([clock_gettime], [], [OBJS="clock_gettime.o"])
AC_CHECK_FUNC([clock_nanosleep], [], [OBJS="$OBJS clock_nanosleep.o"])
AC_CHECK_DECLS([clock_nanosleep],[build_saw_plugin=yes],[build_saw_plugin=no],[#include <time.h>])
AM_CONDITIONAL([BUILD_SAW_EXAMPLE], [test "x$build_saw_plugin" = xyes])

# Test whether the core library should be build
save_LIBS=$LIBS
AC_ARG_ENABLE([corelib-build], AC_HELP_STRING([--enable-corelib-build],
	                [Build the core library. Disabling it may be useful
			when only some plugins must be rebuilt @<:@default=yes@:>@]),
	    [], [enable_corelib_build=yes])
AS_IF([test "x$enable_corelib_build" = xno], 
      [AC_SEARCH_LIBS([egd_open], [eegdev], [], 
        [AC_MSG_FAILURE([eegdev library required since it is not built])])],
      [AS_IF([test "x$HAS_FLEX" = xyes && test "x$HAS_BISON" = xyes],
             [enable_corelib_build=yes],
	     [AC_MSG_ERROR([flex and bison needed to build the core lib])])])
AM_CONDITIONAL([BUILD_CORELIB], [test "x$enable_corelib_build" = xyes])
AC_SUBST([CORE_EEGDEV_LIBS], "$LIBS")
LIBS=$save_LIBS



# EEGfile support
save_LIBS=$LIBS
AC_ARG_WITH([xdf], AC_HELP_STRING([--with-xdf],
	               [Support for EEGfile backend @<:@default=check@:>@]),
	    [], [with_xdf=check])
AS_IF([test "x$with_xdf" != xno], 
     [AC_SEARCH_LIBS([xdf_open], [xdffileio], [xdf_support=yes],
        [xdf_support=no; if test "x$with_xdf" != xcheck; then
            AC_MSG_FAILURE([xdffileio library required for EEGfile support])
         fi])],
     [xdf_support=no])
AM_CONDITIONAL([XDF_SUPPORT], [test "x$xdf_support" = "xyes"])
AS_IF([test "x$xdf_support" = "xyes"], optional_examples=recinxdf$EXEEXT)
AC_SUBST([XDF_LIBS], "$LIBS")
LIBS=$save_LIBS



# Biosemi support
save_LIBS=$LIBS
AC_ARG_WITH([act2], AC_HELP_STRING([--with-act2],
	               [Support for Biosemi backend @<:@default=check@:>@]),
	    [], [with_act2=check])
AS_IF([test "x$with_act2" != xno],
    [PKG_CHECK_MODULES(LIBUSB, [libusb-1.0], [act2_support=yes],
        [act2_support=no; if test "x$with_act2" != xcheck; then
             AC_MSG_FAILURE([libusb-1.0 library required for Biosemi support])
         fi])],
    [act2_support=no])
LIBS="$LIBUSB_LIBS $LIBS"
AS_IF([test "x$act2_support" = xyes], 
      [AC_CHECK_FUNC([libusb_fill_bulk_transfer],
                     [AC_DEFINE([NO_LIBUSB_INLINE_HELPER], [1], [Define to 1 if there is no inline helper function])])])
AM_CONDITIONAL([ACT2_SUPPORT], [test "x$act2_support" = xyes])
AC_SUBST([ACT2_LIBS], "$LIBS")
AC_SUBST([CPPFLAGS], "$LIBUSB_CFLAGS $CPPFLAGS")
LIBS=$save_LIBS



# gTec support
save_LIBS=$LIBS
AC_ARG_WITH([gtec], AC_HELP_STRING([--with-gtec],
		         [Support for gTec backend @<:@default=check@:>@]),
	    [], [with_gtec=check])
gtec_support=no
AS_IF([test "x$with_gtec" != xno],
      [AC_SEARCH_LIBS([GT_OpenDevice], [gUSBampAPI gusbampapi], [gtec_support=yes])])
AS_IF([test "x$with_gtec" != xcheck && test "x$with_gtec" != xno],
      AS_IF([test "x$gtec_support" = xno], 
            AC_MSG_FAILURE([gUSBampAPI library required for gTec support])))
AM_CONDITIONAL([GTEC_SUPPORT], [test "x$gtec_support" = xyes])
AC_SUBST([GTEC_LIBS], "$LIBS")
LIBS=$save_LIBS



# Neurosky support
save_LIBS=$LIBS
AC_ARG_WITH([neurosky], AC_HELP_STRING([--with-neurosky],
	                [Support for Neurosky backend @<:@default=no@:>@]),
	    [], [with_neurosky=no])
AS_IF([test "x$with_neurosky" != xno],
      AC_SEARCH_LIBS([str2ba], [bluetooth], [neurosky_support=yes],
        [neurosky_support=no; if test "x$with_neurosky" != xcheck; then
             AC_MSG_FAILURE([bluetooth library required for neurosky support])
         fi]), [neurosky_support=no])
AM_CONDITIONAL([NSKY_SUPPORT], [test "x$neurosky_support" = xyes])
AC_SUBST([NSKY_LIBS], "$LIBS")
LIBS=$save_LIBS



# Tobi interface A support
save_LIBS=$LIBS
AC_ARG_WITH([tia], AC_HELP_STRING([--with-tia],
	      [Support for Tobi interface A backend @<:@default=check@:>@]),
	    [], [with_tia=check])
AS_IF([test "x$with_tia" != xno],
    [AC_SEARCH_LIBS([XML_Parse], [expat], [tia_support=yes],
        [tia_support=no; if test "x$with_tia" != xcheck; then
             AC_MSG_FAILURE([the expat library is required for Tobi interface A support])
         fi])], [tia_support=no])
AM_CONDITIONAL([TIA_SUPPORT], [test "x$tia_support" = xyes])
AC_SUBST([TIA_LIBS], "$LIBS")
LIBS=$save_LIBS


# BrainProducts RDA (Recorder/RecView) support
save_LIBS=$LIBS
AC_ARG_WITH([barv], AC_HELP_STRING([--with-barv],
          [Support BrainProducts RDA (Recorder/RecView) backend @<:@default=check@:>@]),
        [], [with_barv=check])
AS_IF([test "x$with_barv" != xno],
    [AC_SEARCH_LIBS([pthread_create], [pthread], [barv_support=yes],
        [barv_support=no; if test "x$with_barv" != xcheck; then
             AC_MSG_FAILURE([the pthread library is required for BrainProducts RDA (Recorder/RecView) support])
         fi])], [barv_support=no])
AM_CONDITIONAL([BARV_SUPPORT], [test "x$barv_support" = xyes])
AC_SUBST([BARV_LIBS], "$LIBS")
LIBS=$save_LIBS

# Neurone support
save_LIBS=$LIBS
AC_ARG_WITH([neurone], AC_HELP_STRING([--with-neurone],
          [Support BrainProducts RDA (Recorder/RecView) backend @<:@default=check@:>@]),
        [], [with_neurone=check])
AS_IF([test "x$with_neurone" != xno],
    [AC_SEARCH_LIBS([pthread_create], [pthread], [neurone_support=yes],
        [neurone_support=no; if test "x$with_neurone" != xcheck; then
             AC_MSG_FAILURE([the pthread library is required for BrainProducts RDA (Recorder/RecView) support])
         fi])], [neurone_support=no])
AM_CONDITIONAL([NEURONE_SUPPORT], [test "x$neurone_support" = xyes])
AC_SUBST([NEURONE_LIBS], "$LIBS")
LIBS=$save_LIBS


# BBT support
save_LIBS=$LIBS
AC_ARG_WITH([bbt], AC_HELP_STRING([--with-bbt],
	                [Support for BBT backend @<:@default=no@:>@]),
	    [], [with_bbt=no])
AS_IF([test "x$with_bbt" != xno],
      AC_SEARCH_LIBS([str2ba], [bluetooth], [bbt_support=yes],
        [bbt_support=no; if test "x$with_bbt" != xcheck; then
             AC_MSG_FAILURE([bluetooth library required for BBT support])
         fi]), [bbt_support=no])
AM_CONDITIONAL([BBT_SUPPORT], [test "x$bbt_support" = xyes])
AC_SUBST([BBT_LIBS], "$LIBS")
LIBS=$save_LIBS

# gTecNet support
save_LIBS=$LIBS
AC_ARG_WITH([gtecnet], AC_HELP_STRING([--with-gtecnet],
	                [Support for gTecNet backend @<:@default=no@:>@]),
	    [], [with_gtecnet=no])
AS_IF([test "x$with_gtecnet" != xno],
      AC_SEARCH_LIBS([gtecnal_StartAcquisition], [gtecnalang], [gtecnet_support=yes],
        [gtecnet_support=no; if test "x$with_gtecnet" != xcheck; then
             AC_MSG_FAILURE([libgtecnalang library required for gtecnet support])
         fi]), [gtecnet_support=no])
AM_CONDITIONAL([GTECNET_SUPPORT], [test "x$gtecnet_support" = xyes])
AC_SUBST([GTECNET_LIBS], "$LIBS")
LIBS=$save_LIBS

# LSL support
save_LIBS=$LIBS
AC_ARG_WITH([lsl], AC_HELP_STRING([--with-lsl],
	                [Support for LSL backend @<:@default=no@:>@]),
	    [], [with_lsl=no])
AS_IF([test "x$with_lsl" != xno],
      AC_SEARCH_LIBS([lsl_pull_sample_f], [lsl], [lsl_support=yes],
        [lsl_support=no; if test "x$with_lsl" != xcheck; then
             AC_MSG_FAILURE([lsl library required for LSL support])
         fi]), [lsl_support=no])
AM_CONDITIONAL([LSL_SUPPORT], [test "x$lsl_support" = xyes])
AC_SUBST([LSL_LIBS], "$LIBS")
LIBS=$save_LIBS

# Arduino support
save_LIBS=$LIBS
AC_ARG_WITH([arduino], AC_HELP_STRING([--with-arduino],
	                [Support for Arduino backend @<:@default=no@:>@]),
	    [], [with_arduino=no])
AS_IF([test "x$with_arduino" != xno],
      AC_SEARCH_LIBS([pthread_create], [pthread], [arduino_support=yes],
        [arduino_support=no; if test "x$with_arduino" != xcheck; then
             AC_MSG_FAILURE([pthread library required for arduino support])
         fi]), [arduino_support=no])
AM_CONDITIONAL([ARDUINO_SUPPORT], [test "x$arduino_support" = xyes])
AC_SUBST([ARDUINO_LIBS], "$LIBS")
LIBS=$save_LIBS

# Wearable sensing DSI support
save_LIBS=$LIBS
AC_ARG_WITH([wsdsi], AC_HELP_STRING([--with-wsdsi],
	                [Support for Wearable sensing DSI backend @<:@default=no@:>@]),
	    [], [with_wsdsi=no])
AS_IF([test "x$with_wsdsi" != xno],
      AC_SEARCH_LIBS([DSI_Headset_Connect], [dsi], [wsdsi_support=yes],
        [wsdsi_support=no; if test "x$with_wsdsi" != xcheck; then
             AC_MSG_FAILURE([DSI library required for Wearable Sensing DSI-24 support])
         fi]), [wsdsi_support=no])
AM_CONDITIONAL([WSDSI_SUPPORT], [test "x$wsdsi_support" = xyes])
AC_SUBST([WSDSI_LIBS], "$LIBS")
LIBS=$save_LIBS

# Cognionics Quick-20 support
save_LIBS=$LIBS
AC_ARG_WITH([q20], AC_HELP_STRING([--with-q20],
	                [Support for Cognionics Quick-20 backend @<:@default=no@:>@]),
	    [], [with_q20=no])
AS_IF([test "x$with_q20" != xno],
      AC_SEARCH_LIBS([FT_Open], [ftd2xx], [q20_support=yes],
        [q20_support=no; if test "x$with_q20" != xcheck; then
             AC_MSG_FAILURE([libftd2xx library required for q20 support])
         fi]), [q20_support=no])
AS_IF([test "x$with_q20" != xno],
      AC_SEARCH_LIBS([dlopen], [dl], [q20_support=yes],
        [q20_support=no; if test "x$with_q20" != xcheck; then
             AC_MSG_FAILURE([libdl library required for q20 support])
         fi]), [q20_support=no])
AM_CONDITIONAL([Q20_SUPPORT], [test "x$q20_support" = xyes])
AC_SUBST([Q20_LIBS], "$LIBS")
LIBS=$save_LIBS

# Antneuro eego support
save_LIBS=$LIBS
AC_ARG_WITH([eego], AC_HELP_STRING([--with-eego],
                    [Support for Antneuro eego backend @<:@default=no@:>@]),
        [], [with_eego=no])
AS_IF([test "x$with_eego" != xno],
      AC_SEARCH_LIBS([eemagine_sdk_init], [eego-SDK], [eego_support=yes],
        [eego_support=no; if test "x$with_eego" != xcheck; then
             AC_MSG_FAILURE([eego-SDK library required for Antneuro eego support])
         fi]), [eego_support=no])
AS_IF([test "x$with_eego" != xno],
      AC_SEARCH_LIBS([dlopen], [dl], [eego_support=yes],
        [eego_support=no; if test "x$with_eego" != xcheck; then
             AC_MSG_FAILURE([libdl library required for eego support])
         fi]), [eego_support=no])
AM_CONDITIONAL([EEGO_SUPPORT], [test "x$eego_support" = xyes])
AC_SUBST([EEGO_LIBS], "$LIBS")
LIBS=$save_LIBS


AX_DEFINE_DIR([LIBDIR], [libdir], [Installed library path])
AX_DEFINE_DIR([SYSCONFDIR], [sysconfdir], [System configuration files path])

AC_SUBST([optional_examples])
AC_SUBST([CURRENT],[lib_current])
AC_SUBST([REVISION],[lib_revision])
AC_SUBST([AGE],[lib_age])

# Add ifndef in config.h to fix pthread-win32 header mess
AH_TOP([#ifndef CONFIG_H
#define CONFIG_H])
AH_BOTTOM([#endif /*CONFIG_H*/])

#AC_CONFIG_FILES([Makefile lib/Makefile gnulib-local/Makefile
#                 src/core/Makefile src/core/eegdev.pc
#		src/plugins/Makefile 
#        tests/Makefile tests/fakelibs/Makefile
#		doc/Makefile doc/egd_open.3
#		doc/examples/Makefile
#		doc/examples/library-usage/Makefile])

AC_CONFIG_FILES([Makefile lib/Makefile gnulib-local/Makefile
                 src/core/Makefile src/core/eegdev.pc
		src/plugins/Makefile 
		doc/Makefile doc/egd_open.3])
AC_OUTPUT

echo "

Configuration summary
=====================
    Core library build : $enable_corelib_build
    --------------------------
    EEG file support : $xdf_support
    Biosemi support  : $act2_support
    gTec support     : $gtec_support
    Neurosky support : $neurosky_support
    TobiIA support   : $tia_support
    BARV support     : $barv_support
    NeurOne support  : $neurone_support
    BBT support	     : $bbt_support
    gTecNet support  : $gtecnet_support
    LSL support      : $lsl_support
    Arduino support  : $arduino_support
    Wearable Sensing DSI support  : $wsdsi_support
    Cognionics Quick-20 support  : $q20_support
    Antneuro eego support  : $eego_support
"
