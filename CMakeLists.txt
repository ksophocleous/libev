cmake_minimum_required (VERSION 2.8)

project(libev C)

include("stark.cmake" OPTIONAL RESULT_VARIABLE STARK_FOUND)

include(CheckIncludeFiles)
include(CheckFunctionExists)
include(CheckLibraryExists)

set(package_name "libev")
set(package_bugreport "")
set(package_bugreport "")
set(libtool_objdir ".libs")

set(libname "libev")
set(version_major "4")
set(version_minor "15")

# check headers
check_include_files(inttypeas.h HAVE_INTTYPES_H)
check_include_files(dlfcn.h HAVE_DLFCN_H)
check_include_files(memory.h HAVE_MEMORY_H)
check_include_files(poll.h HAVE_POLL_H)
check_include_files(port.h HAVE_PORT_H)
check_include_files(stdint.h HAVE_STDINT_H)
check_include_files(stdlib.h HAVE_STDLIB_H)
check_include_files(strings.h HAVE_STRINGS_H)
check_include_files(string.h HAVE_STRING_H)
check_include_files(sys/epoll.h HAVE_SYS_EPOLL_H)
check_include_files(sys/eventfd.h HAVE_SYS_EVENTFD_H)
check_include_files(sys/event.h HAVE_SYS_EVENT_H)
check_include_files(sys/inotify.h HAVE_SYS_INOTIFY_H)
check_include_files(sys/select.h HAVE_SYS_SELECT_H)
check_include_files(sys/signalfd.h HAVE_SYS_SIGNALFD_H)
check_include_files(sys/stat.h HAVE_SYS_STAT_H)
check_include_files(sys/types.h HAVE_SYS_TYPES_H)
check_include_files(unistd.h HAVE_UNISTD_H)
check_include_files("assert.h;ctype.h;errno.h;float.h;limits.h;locale.h;math.h;setjmp.h;signal.h;stdarg.h;stddef.h;stdio.h;stdlib.h;string.h;time.h;" STDC_HEADERS)
check_include_files("winsock.h" HAVE_WINSOCK_H)

# check functions
check_function_exists(clock_gettime HAVE_CLOCK_GETTIME)
check_function_exists(epoll_ctl HAVE_EPOLL_CTL)
check_function_exists(eventfd HAVE_EVENTFD)
check_function_exists(floor HAVE_FLOOR)
check_function_exists(inotify_init HAVE_INOTIFY_INIT)
check_function_exists(kqueue HAVE_KQUEUE)
check_function_exists(nanosleep HAVE_NANOSLEEP)
check_function_exists(poll HAVE_POLL)
check_function_exists(port_create HAVE_PORT_CREATE)
check_function_exists(select HAVE_SELECT)
check_function_exists(signalfd HAVE_SIGNALFD)

# check system library
check_library_exists(rt pthread_create "" HAVE_LIBRT)

configure_file(config.h.cmake.in ${CMAKE_CURRENT_BINARY_DIR}/config.h)

# Ignore deprecation warnings and cast warnings
if(MSVC)
	#list(APPEND symbols_warnings "/wd4244" "/wd4996")
	# add_definitions("/wd4244")
	# add_definitions("/wd4996")
endif()

# libev doesn't do dll export so shared wont work without modifying the code
set(libtype STATIC)

set(src
	"ev.c"
	"event.c")

if(WIN32)
	list(APPEND platformlibs "wsock32.lib" "ws2_32.lib")
endif()

set(internal_hdrs
	"${CMAKE_CURRENT_BINARY_DIR}/config.h"
	"ev_wrap.h"
	"event.h"
	"ev_vars.h")

set(public_hdrs "ev.h" "ev++.h")

add_library(${libname} ${libtype} ${src} ${public_hdrs} ${internal_hdrs})
target_link_libraries(${libname} ${platformlibs} ${libs})

set_target_properties(${libname} PROPERTIES 
	INCLUDE_DIRECTORIES "${CMAKE_CURRENT_BINARY_DIR};${CMAKE_CURRENT_SOURCE_DIR}"
	PUBLIC_HEADER "${public_hdrs}"
	STARK_PUBLIC_HEADER_LOC "include"
	STARK_PUBLIC_HEADER_DESTINATION "include"
	STARK_PUBLIC_HEADER_CHECK_FILE "ev.h"
	VERSION 4.15
	PREFIX ""
	LIBFILENAME "ev")

if(MSVC)
	set_target_properties(${libname} PROPERTIES
		COMPILE_FLAGS "/wd4244 /wd4996")
endif()

if (STARK_FOUND)
	stark_install(${libname})
	stark_unify(${libname})
endif()
