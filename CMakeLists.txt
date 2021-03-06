# This file is part of GUFI, which is part of MarFS, which is released
# under the BSD license.
#
#
# Copyright (c) 2017, Los Alamos National Security (LANS), LLC
# All rights reserved.
#
# Redistribution and use in source and binary forms, with or without modification,
# are permitted provided that the following conditions are met:
#
# 1. Redistributions of source code must retain the above copyright notice, this
# list of conditions and the following disclaimer.
#
# 2. Redistributions in binary form must reproduce the above copyright notice,
# this list of conditions and the following disclaimer in the documentation and/or
# other materials provided with the distribution.
#
# 3. Neither the name of the copyright holder nor the names of its contributors
# may be used to endorse or promote products derived from this software without
# specific prior written permission.
#
# THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS" AND
# ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED
# WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE DISCLAIMED.
# IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT,
# INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING,
# BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE,
# DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF
# LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE
# OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF
# ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
#
# -----
# NOTE:
# -----
#
# GUFI uses the C-Thread-Pool library.  The original version, written by
# Johan Hanssen Seferidis, is found at
# https://github.com/Pithikos/C-Thread-Pool/blob/master/LICENSE, and is
# released under the MIT License.  LANS, LLC added functionality to the
# original work.  The original work, plus LANS, LLC added functionality is
# found at https://github.com/jti-lanl/C-Thread-Pool, also under the MIT
# License.  The MIT License can be found at
# https://opensource.org/licenses/MIT.
#
#
# From Los Alamos National Security, LLC:
# LA-CC-15-039
#
# Copyright (c) 2017, Los Alamos National Security, LLC All rights reserved.
# Copyright 2017. Los Alamos National Security, LLC. This software was produced
# under U.S. Government contract DE-AC52-06NA25396 for Los Alamos National
# Laboratory (LANL), which is operated by Los Alamos National Security, LLC for
# the U.S. Department of Energy. The U.S. Government has rights to use,
# reproduce, and distribute this software.  NEITHER THE GOVERNMENT NOR LOS
# ALAMOS NATIONAL SECURITY, LLC MAKES ANY WARRANTY, EXPRESS OR IMPLIED, OR
# ASSUMES ANY LIABILITY FOR THE USE OF THIS SOFTWARE.  If software is
# modified to produce derivative works, such modified software should be
# clearly marked, so as not to confuse it with the version available from
# LANL.
#
# THIS SOFTWARE IS PROVIDED BY LOS ALAMOS NATIONAL SECURITY, LLC AND CONTRIBUTORS
# "AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO,
# THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE
# ARE DISCLAIMED. IN NO EVENT SHALL LOS ALAMOS NATIONAL SECURITY, LLC OR
# CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL,
# EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT
# OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS
# INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN
# CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING
# IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY
# OF SUCH DAMAGE.



cmake_minimum_required(VERSION 3.0.0)
project(GUFI C)

# Require C99
set(CMAKE_C_STANDARD 99)
set(CMAKE_C_STANDARD_REQUIRED ON)

# define the DEBUG macro when building in debug mode
set(CMAKE_C_FLAGS_DEBUG "${CMAKE_C_FLAGS_DEBUG} -DDEBUG -Wall -D_XOPEN_SRC")

# only enable C++ if support is found
include(CheckLanguage)
check_language(CXX)
if (CMAKE_CXX_COMPILER)
  enable_language(CXX)

  # GCC prior to 4.9.3 doesn't have full standard support
  if ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU" AND CMAKE_CXX_COMPILER_VERSION VERSION_LESS 4.9.3)
    set(CMAKE_CXX_COMPILER FALSE)
    message(STATUS "Require GNU C++ version 4.9.3 or greater: Not compiling C++ sources")
  elseif ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "Clang" AND CMAKE_CXX_COMPILER_VERSION VERSION_LESS 3.9)
    set(CMAKE_CXX_COMPILER FALSE)
    message(STATUS "Require Clang version 3.9 or greater: not compiling C++ sources")
  else()
    # enable C++11
    set(CMAKE_CXX_STANDARD 11)
    set(CMAKE_CXX_STANDARD_REQUIRED ON)
    set(CMAKE_CXX_EXTENSIONS OFF)

    # define the DEBUG macro when building in debug mode
    set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -DDEBUG -Wall")
  endif()
else()
  set(CMAKE_CXX_COMPILER FALSE)
  message(STATUS "Not compiling C++ sources")
endif()

# Re-link installed executables to use the installed libraries
set(CMAKE_SKIP_BUILD_RPATH TRUE)
set(CMAKE_BUILD_WITH_INSTALL_RPATH TRUE)

# need the PkgConfig module for pkg_search_module
find_package(PkgConfig REQUIRED)

# tell CMake to search the contrib/cmake directory cmake modules
set(CMAKE_MODULE_PATH ${CMAKE_SOURCE_DIR}/contrib/cmake ${CMAKE_MODULE_PATH})

# Detect and handle xattr.h platform specifics
find_package(xattr REQUIRED XATTR)
message(STATUS "xattr headers located in ${XATTR_INCLUDEDIR}")
if (XATTR_ATTR_XATTR_HEADER)
   set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -DCONFIG_ATTR_XATTR_H")
   set(CMAKE_C_FLAGS_DEBUG "${CMAKE_C_FLAGS_DEBUG} -DCONFIG_ATTR_XATTR_H")
   set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DCONFIG_ATTR_XATTR_H")
   set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -DCONFIG_ATTR_XATTR_H")
endif()
if (XATTR_SYS_XATTR_HEADER)
   set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -DCONFIG_SYS_XATTR_H")
   set(CMAKE_C_FLAGS_DEBUG "${CMAKE_C_FLAGS_DEBUG} -DCONFIG_SYS_XATTR_H")
   set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DCONFIG_SYS_XATTR_H")
   set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -DCONFIG_SYS_XATTR_H")
endif()
if (APPLE)
   set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -DCONFIG_DARWIN_XATTRS")
   set(CMAKE_C_FLAGS_DEBUG "${CMAKE_C_FLAGS_DEBUG} -DCONFIG_DARWIN_XATTRS")
   set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DCONFIG_DARWIN_XATTRS")
   set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -DCONFIG_DARWIN_XATTRS")
else()
   set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -DCONFIG_GNU_XATTRS")
   set(CMAKE_C_FLAGS_DEBUG "${CMAKE_C_FLAGS_DEBUG} -DCONFIG_GNU_XATTRS")
   set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DCONFIG_GNU_XATTRS")
   set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -DCONFIG_GNU_XATTRS")
endif()

# need pthread
find_package(Threads REQUIRED)

# load the ExternalProject module
include(ExternalProject)

# Allow the user to chose to use HTTPS or SSH
set(GIT_DL_METHOD HTTPS CACHE STRING "Use HTTPS or SSH to download git repositories")
string(TOLOWER "${GIT_DL_METHOD}" GIT_DL_METHOD)

# download and build C-Thread-Pool
set(thpool_PREFIX ${CMAKE_BINARY_DIR}/src)
if ("${GIT_DL_METHOD}" STREQUAL "https")
  ExternalProject_Add(C-Thread-Pool
    GIT_REPOSITORY    https://github.com/mar-file-system/C-Thread-Pool.git
    GIT_TAG           lanl
    GIT_SHALLOW       1
    PREFIX            ${CMAKE_BINARY_DIR}/
    STEP_TARGETS      build
    INSTALL_COMMAND   ""
  )
elseif ("${GIT_DL_METHOD}" STREQUAL "ssh")
  ExternalProject_Add(C-Thread-Pool
    GIT_REPOSITORY    git@github.com:mar-file-system/C-Thread-Pool.git
    GIT_TAG           lanl
    GIT_SHALLOW       1
    PREFIX            ${CMAKE_BINARY_DIR}/
    STEP_TARGETS      build
    INSTALL_COMMAND   ""
  )
endif()

# Find SQLite3
pkg_search_module(SQLITE3 REQUIRED sqlite3)
if (SQLITE3_VERSION VERSION_LESS 3.13)
  message(FATAL_ERROR "GUFI requires sqlite3 3.13+. Got ${SQLITE3_VERSION}")
endif()
include_directories(AFTER SYSTEM ${SQLITE3_INCLUDE_DIRS}) # -Ipath/to/sqlite3/include
link_directories(${SQLITE3_LIBRARY_DIRS})                 # -Lpath/to/sqlite3/lib
message(STATUS "SQLite3 headers located in ${SQLITE3_INCLUDEDIR}")
message(STATUS "SQLite3 libraries located in ${SQLITE3_LIBRARY_DIRS}")
set(CMAKE_INSTALL_RPATH "${SQLITE3_LIBRARY_DIRS}")

# download and build sqlite3-pcre
pkg_search_module(PCRE REQUIRED libpcre)
set(pcre_PREFIX     ${CMAKE_BINARY_DIR}/src/sqlite3-pcre)
if ("${GIT_DL_METHOD}" STREQUAL "https")
  ExternalProject_Add(sqlite3-pcre
    GIT_REPOSITORY    https://github.com/mar-file-system/sqlite3-pcre.git
    GIT_TAG           master
    GIT_SHALLOW       1
    PREFIX            ${CMAKE_BINARY_DIR}/
    STEP_TARGETS      build
    INSTALL_COMMAND   ""
  )
elseif ("${GIT_DL_METHOD}" STREQUAL "ssh")
  ExternalProject_Add(sqlite3-pcre
    GIT_REPOSITORY    git@github.com:mar-file-system/sqlite3-pcre.git
    GIT_TAG           master
    GIT_SHALLOW       1
    PREFIX            ${CMAKE_BINARY_DIR}/
    STEP_TARGETS      build
    INSTALL_COMMAND   ""
  )
endif()

# search for FUSE on Linux, or OSXFUSE on OSX
if (UNIX AND NOT APPLE AND NOT CYGWIN)
  pkg_search_module(FUSE fuse)
  if (FUSE_FOUND)
    message(STATUS "FUSE headers located in ${FUSE_INCLUDEDIR}")
    message(STATUS "FUSE libraries located in ${FUSE_LIBDIR}")

    set(MY_FUSE_FOUND YES)
    set(MY_FUSE_CFLAGS ${FUSE_CFLAGS})
    set(MY_FUSE_LDFLAGS ${FUSE_LDFLAGS})
  endif()
elseif (APPLE)
  pkg_search_module(OSXFUSE osxfuse)
  if (OSXFUSE_FOUND)
    message(STATUS "OSXFUSE headers located in ${OSXFUSE_INCLUDEDIR}")
    message(STATUS "OSXFUSE libraries located in ${OSXFUSE_LIBDIR}")

    set(MY_FUSE_FOUND YES)
    set(MY_FUSE_CFLAGS ${OSXFUSE_CFLAGS})
    set(MY_FUSE_LDFLAGS ${OSXFUSE_LDFLAGS})
  endif()
endif()

if (NOT MY_FUSE_FOUND)
  message(STATUS "FUSE not found. Not building bffuse and bfresultfuse.")
endif()

# Find MySQL
pkg_search_module(MYSQL mysql)
if (MYSQL_FOUND)
  message(STATUS "MySQL headers located in ${MYSQL_INCLUDEDIR}")
  message(STATUS "MySQL libraries located in ${MYSQL_LIBDIR}")
else()
  message(STATUS "MySQL not found. Not building bfmi.")
endif()

# Find DB2
pkg_search_module(DB2 db2)
if (DB2_FOUND)
  message(STATUS "DB2 headers located in ${DB2_INCLUDEDIR}")
  message(STATUS "DB2 libraries located in ${DB2_LIBDIR}")
else()
  message(STATUS "DB2 not found.")
endif()

# everyone includes (-I) these paths, whether or not they need them
set(COMMON_INCLUDES ${CMAKE_SOURCE_DIR}/include ${XATTR_INCLUDEDIR} ${UUID_INCLUDEDIR} ${SQLITE3_INCLUDEDIR} ${pcre_PREFIX} ${thpool_PREFIX})
include_directories(${COMMON_INCLUDES})

# set of libraries needed by the final binaries
set(COMMON_LIBRARIES GUFI ${pcre_PREFIX}-build/libsqlite3-pcre.a sqlite3 pcre ${thpool_PREFIX}/C-Thread-Pool-build/libthpool.a Threads::Threads m)

# tell CMake to look at the src directory
add_subdirectory(src)

# process the scripts directory
add_subdirectory(scripts)

# enable "make test"
enable_testing()

# recurse down into the test subdirectory
add_subdirectory(test)

#
# Define commands and tools for "make gary"
# build tarball out of ${CMAKE_SOURCE_DIR}
#

# collect all files into variables
set(OTHER_FILES
  CMakeLists.txt
  INSTALL
  LICENSE.txt
  Makefile
  README.md)
set(PATHS
  contrib
  docs
  examples
  include
  scripts
  src
  test
  C-Thread-Pool
  sqlite3-pcre
  googletest)

# set the output nemae
set(TARBALL_NAME "${CMAKE_BINARY_DIR}/gufi.tar.gz")

add_custom_target(cp_files COMMAND ${CMAKE_COMMAND} -E copy ${CMAKE_SOURCE_DIR}/contrib/make/root ${CMAKE_SOURCE_DIR}/Makefile
                           COMMAND ${CMAKE_COMMAND} -E copy ${CMAKE_SOURCE_DIR}/contrib/make/src ${CMAKE_SOURCE_DIR}/src/Makefile
                           COMMAND ${CMAKE_COMMAND} -E copy ${CMAKE_SOURCE_DIR}/contrib/make/scripts ${CMAKE_SOURCE_DIR}/scripts/Makefile
                           COMMAND ${CMAKE_COMMAND} -E copy ${CMAKE_SOURCE_DIR}/contrib/make/test ${CMAKE_SOURCE_DIR}/test/Makefile
                           COMMAND ${CMAKE_COMMAND} -E copy ${CMAKE_SOURCE_DIR}/contrib/make/googletest ${CMAKE_SOURCE_DIR}/test/googletest/Makefile
                           COMMAND ${CMAKE_COMMAND} -E copy_directory ${CMAKE_BINARY_DIR}/src/C-Thread-Pool ${CMAKE_SOURCE_DIR}/C-Thread-Pool
                           COMMAND ${CMAKE_COMMAND} -E copy_directory ${CMAKE_BINARY_DIR}/src/sqlite3-pcre ${CMAKE_SOURCE_DIR}/sqlite3-pcre
                           COMMAND ${CMAKE_COMMAND} -E copy_directory ${CMAKE_BINARY_DIR}/googletest-src ${CMAKE_SOURCE_DIR}/googletest
                           COMMAND ${CMAKE_COMMAND} -E remove_directory ${CMAKE_SOURCE_DIR}/C-Thread-Pool/.git
                           COMMAND ${CMAKE_COMMAND} -E remove_directory ${CMAKE_SOURCE_DIR}/sqlite3-pcre/.git
                           COMMAND ${CMAKE_COMMAND} -E remove_directory ${CMAKE_SOURCE_DIR}/googletest/.git
                           DEPENDS C-Thread-Pool sqlite3-pcre test/googletest)

# generate the tarball
add_custom_target(create_tarball COMMAND ${CMAKE_COMMAND} -E tar "czf" ${TARBALL_NAME} ${PATHS} ${TOOLS} ${OTHER_FILES} WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
                                 DEPENDS cp_files)

# the actual target
add_custom_target(gary COMMAND ${CMAKE_COMMAND} -E remove ${CMAKE_SOURCE_DIR}/Makefile
                       COMMAND ${CMAKE_COMMAND} -E remove ${CMAKE_SOURCE_DIR}/src/Makefile
                       COMMAND ${CMAKE_COMMAND} -E remove ${CMAKE_SOURCE_DIR}/scripts/Makefile
                       COMMAND ${CMAKE_COMMAND} -E remove ${CMAKE_SOURCE_DIR}/test/Makefile
                       COMMAND ${CMAKE_COMMAND} -E remove ${CMAKE_SOURCE_DIR}/test/googletest/Makefile
                       COMMAND ${CMAKE_COMMAND} -E remove_directory ${CMAKE_SOURCE_DIR}/C-Thread-Pool
                       COMMAND ${CMAKE_COMMAND} -E remove_directory ${CMAKE_SOURCE_DIR}/sqlite3-pcre
                       COMMAND ${CMAKE_COMMAND} -E remove_directory ${CMAKE_SOURCE_DIR}/googletest
                       DEPENDS create_tarball)

# test the tarball
set(TARBALL_TEST_DIR "${CMAKE_BINARY_DIR}/tarball")

add_custom_target(make_tarball_dir COMMAND ${CMAKE_COMMAND} -E remove_directory ${TARBALL_TEST_DIR}
                                   COMMAND ${CMAKE_COMMAND} -E make_directory ${TARBALL_TEST_DIR})

add_custom_target(build_gary COMMAND ${CMAKE_COMMAND} -E tar "xzf" ${TARBALL_NAME}
                             COMMAND make DEBUG=1
                             COMMAND make test
                             WORKING_DIRECTORY ${TARBALL_TEST_DIR}
                             DEPENDS gary make_tarball_dir)

add_custom_target(test_gary COMMAND ${CMAKE_COMMAND} -E remove_directory ${TARBALL_TEST_DIR}
                            WORKING_DIRECTORY ${CMAKE_BINARY_DIR}
                            DEPENDS build_gary)
