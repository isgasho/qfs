#
# $Id$
#
# Created 2006
# Author: Sriram Rao (Kosmix Corp)
#
# Copyright 2008-2012,2016 Quantcast Corporation. All rights reserved.
# Copyright 2006 Kosmix Corp.
#
# This file is part of Kosmos File System (KFS).
#
# Licensed under the Apache License, Version 2.0
# (the "License"); you may not use this file except in compliance with
# the License. You may obtain a copy of the License at
#
# http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or
# implied. See the License for the specific language governing
# permissions and limitations under the License.
#
#

# Take all the .cc files and build a library out of them

set (sources
    Acceptor.cc
    checksum.cc
    Globals.cc
    IOBuffer.cc
    NetConnection.cc
    NetErrorSimulator.cc
    NetManager.cc
    TcpSocket.cc
    ZlibInflate.cc
    KfsCallbackObj.cc
    SslFilter.cc
    ClientAuthContext.cc
    DelegationToken.cc
    Base64.cc
    CryptoKeys.cc
    ChunkAccessToken.cc
    PrngIsaac64.cc
    NetForwarder.cc
    TransactionalClient.cc
    HttpResponseHeaders.cc
    HttpChunkedDecoder.cc
    blockname.cc
    ProcessRestarter.cc
    Resolver.cc
)

if (QFS_OMIT_EXT_DNS_RESOLVER)
    add_definitions(-DQFS_OMIT_EXT_DNS_RESOLVER)
else (QFS_OMIT_EXT_DNS_RESOLVER)
    set (dns-src-dir ${KFS_DIR_PREFIX}/ext/dns/src)
    set (sources ${sources}
        ${dns-src-dir}/dns.c
    )
    include_directories (
        ${dns-src-dir}
    )
    add_definitions(-D_GNU_SOURCE)
    if (CMAKE_C_COMPILER_ID STREQUAL "GNU")
        set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -std=c99")
    endif (CMAKE_C_COMPILER_ID STREQUAL "GNU")
endif (QFS_OMIT_EXT_DNS_RESOLVER)

add_library (kfsIO STATIC ${sources})
add_library (kfsIO-shared SHARED ${sources})
set_target_properties (kfsIO PROPERTIES OUTPUT_NAME "qfs_io")
set_target_properties (kfsIO-shared PROPERTIES OUTPUT_NAME "qfs_io")

set_target_properties (kfsIO PROPERTIES CLEAN_DIRECT_OUTPUT 1)
set_target_properties (kfsIO-shared PROPERTIES CLEAN_DIRECT_OUTPUT 1)

add_dependencies (kfsIO
    kfsCommon
    qfskrb
)
add_dependencies (kfsIO-shared
    kfsCommon-shared
    qfskrb-shared
)
target_link_libraries (kfsIO
    kfsCommon
    qfskrb
    z
    ${OPENSSL_LIBRARIES}
)
target_link_libraries (kfsIO-shared
    kfsCommon-shared
    qfskrb-shared
    z
    ${OPENSSL_LIBRARIES}
)

if (CMAKE_SYSTEM_NAME STREQUAL "SunOS")
    target_link_libraries (kfsIO nsl socket resolv)
    target_link_libraries (kfsIO-shared nsl socket resolv)
endif (CMAKE_SYSTEM_NAME STREQUAL "SunOS")

install (TARGETS kfsIO kfsIO-shared
        LIBRARY DESTINATION lib
        ARCHIVE DESTINATION lib/static)
