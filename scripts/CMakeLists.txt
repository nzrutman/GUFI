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

# list of scripts that act as tools
set(TOOLS gufi_find gufi_stats gufi_ls query_builder.py)

set(PARAMIKO OFF CACHE BOOL "Whether or not to download and install paramiko")
if (PARAMIKO)
  # download paramiko
  set(PARAMIKO_PREFIX ${CMAKE_BINARY_DIR}/src)
  if ("${GIT_DL_METHOD}" STREQUAL "https")
    ExternalProject_Add(paramiko
      GIT_REPOSITORY    https://github.com/paramiko/paramiko.git
      GIT_TAG           cf7d49d66f3b1fbc8b0853518a54050182b3b5eb
      GIT_SHALLOW       1
      PREFIX            ${CMAKE_BINARY_DIR}/
      PATCH_COMMAND     git apply ${CMAKE_SOURCE_DIR}/contrib/paramiko-cf7d49d-hostbased.patch
      CONFIGURE_COMMAND ""
      BUILD_COMMAND     ""
      INSTALL_COMMAND   ""
    )
  elseif ("${GIT_DL_METHOD}" STREQUAL "ssh")
    ExternalProject_Add(paramiko
      GIT_REPOSITORY    git@github.com:paramiko/paramiko.git
      GIT_TAG           cf7d49d66f3b1fbc8b0853518a54050182b3b5eb
      GIT_SHALLOW       1
      PREFIX            ${CMAKE_BINARY_DIR}/
      PATCH_COMMAND     git apply ${CMAKE_SOURCE_DIR}/contrib/paramiko-cf7d49d-hostbased.patch
      CONFIGURE_COMMAND ""
      BUILD_COMMAND     ""
      INSTALL_COMMAND   ""
    )
  endif()

  # add gufi_client to the list of tools
  set(TOOLS ${TOOLS} gufi_client)

  # add the paramiko source for installing
  install(DIRECTORY ${CMAKE_BINARY_DIR}/src/paramiko DESTINATION ${CMAKE_INSTALL_PREFIX}/paramiko)
endif()

# copy and install scripts that act like binaries
foreach(TOOL ${TOOLS})
  # copy the tool into the build directory for easy access
  configure_file(${TOOL} ${TOOL} COPYONLY)

  # install these tools into ${CMAKE_INSTALL_PREFIX}/bin
  install(PROGRAMS ${TOOL} DESTINATION bin)
endforeach()
