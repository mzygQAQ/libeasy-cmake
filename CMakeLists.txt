project(libeasy)
cmake_minimum_required(VERSION 3.10)

option(EASY_BUILD_WITH_SSL "whether build with openssl support" OFF)
option(EASY_BUILD_SHARD_LIBRARY "whether build a shard lib instead of a static lib" OFF)
option(EASY_BUILD_TEST "whether build the unittest code" OFF)
option(EASY_BUILD_SAMPLE "whether build the sample code" OFF)

set(EASY_INC 
    "${CMAKE_SOURCE_DIR}/src/include"
    "${CMAKE_SOURCE_DIR}/src/io"
    "${CMAKE_SOURCE_DIR}/src/memory"
    "${CMAKE_SOURCE_DIR}/src/thread"
    "${CMAKE_SOURCE_DIR}/src/util"
    "${CMAKE_SOURCE_DIR}/src/packet"
    "${CMAKE_SOURCE_DIR}/src/packet/http"
)

set(EASY_SRCS 
# util
    "${CMAKE_SOURCE_DIR}/src/util/easy_array.c"
    "${CMAKE_SOURCE_DIR}/src/util/easy_buf.c"
    "${CMAKE_SOURCE_DIR}/src/util/easy_hash.c"
    "${CMAKE_SOURCE_DIR}/src/util/easy_inet.c"
    "${CMAKE_SOURCE_DIR}/src/util/easy_lock.c"
    "${CMAKE_SOURCE_DIR}/src/util/easy_pool.c"
    "${CMAKE_SOURCE_DIR}/src/util/easy_ref.c"
    "${CMAKE_SOURCE_DIR}/src/util/easy_string.c" 
    "${CMAKE_SOURCE_DIR}/src/util/easy_time.c"
# thread
    "${CMAKE_SOURCE_DIR}/src/thread/easy_uthread.c"
# packet
    "${CMAKE_SOURCE_DIR}/src/packet/easy_kfc_handler.c"
    "${CMAKE_SOURCE_DIR}/src/packet/http/easy_http_handler.c"
    "${CMAKE_SOURCE_DIR}/src/packet/http/http_parser.c"
# memory
    "${CMAKE_SOURCE_DIR}/src/memory/easy_mem_page.c"
    "${CMAKE_SOURCE_DIR}/src/memory/easy_mem_pool.c"
    "${CMAKE_SOURCE_DIR}/src/memory/easy_mem_slab.c"
# io
    "${CMAKE_SOURCE_DIR}/src/io/easy_baseth_pool.c"
    "${CMAKE_SOURCE_DIR}/src/io/easy_client.c"
    "${CMAKE_SOURCE_DIR}/src/io/easy_connection.c"
    "${CMAKE_SOURCE_DIR}/src/io/easy_event.c"
    "${CMAKE_SOURCE_DIR}/src/io/easy_file.c"
    "${CMAKE_SOURCE_DIR}/src/io/easy_io.c"
    "${CMAKE_SOURCE_DIR}/src/io/easy_log.c"
    "${CMAKE_SOURCE_DIR}/src/io/easy_message.c"
    "${CMAKE_SOURCE_DIR}/src/io/easy_request.c"
    "${CMAKE_SOURCE_DIR}/src/io/easy_socket.c"
    "${CMAKE_SOURCE_DIR}/src/io/easy_summary.c"
    "${CMAKE_SOURCE_DIR}/src/io/ev.c")

if(${EASY_BUILD_WITH_SSL})
message(STATUS "Building with SSL support")
# TODO: check openssl
set(EASY_SRCS "${EASY_SRCS}" "${CMAKE_SOURCE_DIR}/src/io/easy_ssl.c")
endif()

# for O_ATEXITTIME
add_compile_definitions(_GNU_SOURCE=1)

add_library(easy_OBJS OBJECT ${EASY_SRCS})
target_include_directories(easy_OBJS PUBLIC ${EASY_INC})
if(${EASY_BUILD_SHARD_LIBRARY})
set_target_properties(easy_OBJS PROPERTIES POSITION_INDEPENDENT_CODE ON)
endif()

if(${EASY_BUILD_SHARD_LIBRARY})
add_library(easy_shared SHARED $<TARGET_OBJECTS:easy_OBJS>)
else()
add_library(easy_static STATIC $<TARGET_OBJECTS:easy_OBJS>)
endif()

#TODO: test
#TODO: sample
# TODO: install