
# CyberCache Cluster
# Written by Vadim Sytnikov.
# Copyright (C) 2016-2019 CyberHULL. All rights reserved.
#
# This program is free software: you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation, either version 2 of the License, or
# (at your option) any later version.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the
# GNU General Public License for more details.
# -----------------------------------------------------------------------------
#
# CyberCache PHP Extension module.
#

project(CyberCacheExtension)

# set this to "-DZTS -DPTHREADS" to enable thread safety
set(PHP_THREAD_SAFETY_SWITCH "")
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${PHP_THREAD_SAFETY_SWITCH}")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${PHP_THREAD_SAFETY_SWITCH}")

set(EXTENSION_SOURCE_FILES
    external_apis.h external_apis.cc
    regex_matcher.cc regex_matcher.h
    option_utils.cc option_utils.h
    ext_errors.cc ext_errors.h
    ext_exceptions.cc ext_exceptions.h
    ext_globals.cc ext_globals.h
    ext_functions.h ext_functions.cc
    ext_resource.cc ext_resource.h
    server_thunk.cc server_thunk.h
    extension.h extension.cc)

set(EXTENSION_COMMON_LIBRARIES
    comp_lz4
    comp_lzf
    comp_lzham
    comp_snappy
    comp_zlib
    comp_zstd
    hash_farmhash
    hash_murmurhash
    hash_spookyhash
    hash_xxhash
    ${CMAKE_THREAD_LIBS_INIT})

# PHP APIs against which to build the extension(s): MUST match installed PHP `-dev` packages!
set(PHP_APIS 20151012 20160303 20170718 20180731)
foreach(API IN LISTS PHP_APIS)

# Set include directories
set(PHP_INC_BASE "/usr/include/php/${API}")
set(PHP_INC_DIRS "${PHP_INC_BASE}" "${PHP_INC_BASE}/main" "${PHP_INC_BASE}/TSRM" "${PHP_INC_BASE}/Zend")

# Community Edition build
set(CE_LIB "ext_cybercache_ce_${API}")
add_library(${CE_LIB} SHARED
    ${EXTENSION_SOURCE_FILES})
set_property(TARGET ${CE_LIB} PROPERTY OUTPUT_NAME "cybercache-ce-${API}")
set_property(TARGET ${CE_LIB} PROPERTY PREFIX "")
target_link_libraries(${CE_LIB}
    c3lib_ce
    ${EXTENSION_COMMON_LIBRARIES})
target_include_directories(${CE_LIB} PUBLIC ${PHP_INC_DIRS})

# Enterprise Edition build
set(EE_LIB "ext_cybercache_ee_${API}")
add_library(${EE_LIB} SHARED
    ${EXTENSION_SOURCE_FILES})
set_property(TARGET ${EE_LIB} PROPERTY OUTPUT_NAME "cybercache-ee-${API}")
set_property(TARGET ${EE_LIB} PROPERTY PREFIX "")
target_link_libraries(${EE_LIB}
    c3lib_ee
    comp_brotli
    ${EXTENSION_COMMON_LIBRARIES})
target_include_directories(${EE_LIB} PUBLIC ${PHP_INC_DIRS})

endforeach()
