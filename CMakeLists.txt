cmake_minimum_required(VERSION 3.1)

# extract version out of configure.ac
file(READ "configure.ac" CONFIGURE_AC)
string(REGEX MATCH "AC_INIT\\(\\[libjpeg\\], \\[([0-9.]+)\\]\\)" ver_found "${CONFIGURE_AC}")
if(NOT ver_found)
	message(FATAL_ERROR "Could not extract version from configure.ac")
endif()
set(LIBJPEG_VERSION "${CMAKE_MATCH_1}")

project(jpeg LANGUAGES C VERSION ${LIBJPEG_VERSION})

include(GNUInstallDirs)

add_library(jpeg)
target_sources(jpeg PRIVATE
		jaricom.c jcapimin.c jcapistd.c jcarith.c jccoefct.c jccolor.c
		jcdctmgr.c jchuff.c jcinit.c jcmainct.c jcmarker.c jcmaster.c
		jcomapi.c jcparam.c jcprepct.c jcsample.c jctrans.c jdapimin.c
		jdapistd.c jdarith.c jdatadst.c jdatasrc.c jdcoefct.c jdcolor.c
		jddctmgr.c jdhuff.c jdinput.c jdmainct.c jdmarker.c jdmaster.c
		jdmerge.c jdpostct.c jdsample.c jdtrans.c jerror.c jfdctflt.c
		jfdctfst.c jfdctint.c jidctflt.c jquant1.c
		jquant2.c jutils.c jmemmgr.c)

set(jpeg_headers
	jconfig.h
	jerror.h
	jmorecfg.h
	jpeglib.h)

if (ANDROID)
	target_sources(jpeg PRIVATE jmem-android.c)
else()
	target_sources(jpeg PRIVATE jmemansi.c)
endif()


target_sources(jpeg PRIVATE jidctint.c jidctfst.c)

# FIXME : include asm for ARM
# target_sources(jpeg PRIVATE jidctint.c jidctfst.S)

target_compile_definitions(jpeg PRIVATE AVOID_TABLES)

target_include_directories(jpeg PUBLIC
	$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
	$<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>)

set_target_properties(jpeg PROPERTIES
	SOVERSION "${PROJECT_VERSION_MAJOR}"
	VERSION "${PROJECT_VERSION}"
	PUBLIC_HEADER "${jpeg_headers}"
	# create libjpeg-9.dll on Windows
	RUNTIME_OUTPUT_NAME "jpeg-${PROJECT_VERSION_MAJOR}"
	PREFIX lib)

if(MINGW AND BUILD_SHARED_LIBS)
	set_target_properties(jpeg PROPERTIES
		PREFIX lib)
endif()

install(TARGETS jpeg)
