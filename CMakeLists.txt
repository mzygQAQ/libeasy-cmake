project(libeasy)
cmake_minimum_required(VERSION 3.10)

option(EASY_BUILD_SHARD "whether build a shard lib instead of a static lib" OFF)
option(EASY_BUILD_TEST "whether build the unittest code" OFF)
option(EASY_BUILD_SAMPLE "whether build the sample code" OFF)

# for O_ATEXITTIME
add_compile_definitions(_GNU_SOURCE=1)

# checking openssl
include(CheckIncludeFile)
CHECK_INCLUDE_FILE("openssl/ssl.h" HAVE_OPENSSL_SSL_H)
if(NOT ${HAVE_OPENSSL_SSL_H})
    message(FATAL_ERROR "You have to install OpenSSL(<1.1.0) first!")
endif()

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
    "${CMAKE_SOURCE_DIR}/src/io/easy_ssl.c"
    "${CMAKE_SOURCE_DIR}/src/io/ev.c")


add_library(easy_OBJS OBJECT ${EASY_SRCS})
target_include_directories(easy_OBJS PUBLIC ${EASY_INC})
if(${EASY_BUILD_SHARD})
    # -fPIC
    set_target_properties(easy_OBJS PROPERTIES POSITION_INDEPENDENT_CODE ON)
endif()

if(${EASY_BUILD_SHARD})
    add_library(easy_shared SHARED $<TARGET_OBJECTS:easy_OBJS>)
else()
    add_library(easy_static STATIC $<TARGET_OBJECTS:easy_OBJS>)
endif()


if(${EASY_BUILD_TEST}) 
    set(EASY_TEST_INC 
        "${EASY_INC}"
        "${CMAKE_SOURCE_DIR}/test"
        "${CMAKE_SOURCE_DIR}/test/include"
        "${CMAKE_SOURCE_DIR}/test/io"
        "${CMAKE_SOURCE_DIR}/test/memory"
        "${CMAKE_SOURCE_DIR}/test/packet"
        "${CMAKE_SOURCE_DIR}/test/thread"
        "${CMAKE_SOURCE_DIR}/test/util"
    )

    set(EASY_TEST_LINK_LIB "pthread")
    if(${EASY_BUILD_SHARD})
        list(APPEND EASY_TEST_LINK_LIB "easy_shared")
    else()
        list(APPEND EASY_TEST_LINK_LIB "easy_static")
    endif()

    add_executable(easy_include_test
        "${CMAKE_SOURCE_DIR}/test/include/easy_atomic_test.c"
        "${CMAKE_SOURCE_DIR}/test/include/easy_list_test.c"
        "${CMAKE_SOURCE_DIR}/test/include/include_test.c"
    )
    target_include_directories(easy_include_test PUBLIC ${EASY_TEST_INC})
    target_link_libraries(easy_include_test ${EASY_TEST_LINK_LIB})

    add_executable(easy_util_test
        "${CMAKE_SOURCE_DIR}/test/util/easy_buf_test.c"
        "${CMAKE_SOURCE_DIR}/test/util/easy_hash_test.c"
        "${CMAKE_SOURCE_DIR}/test/util/easy_inet_test.c"
        "${CMAKE_SOURCE_DIR}/test/util/easy_list_test.c"
        "${CMAKE_SOURCE_DIR}/test/util/easy_lock_test.c"
        "${CMAKE_SOURCE_DIR}/test/util/easy_pool_test.c"
        "${CMAKE_SOURCE_DIR}/test/util/easy_ref_test.c"
        "${CMAKE_SOURCE_DIR}/test/util/easy_string_test.c"
        "${CMAKE_SOURCE_DIR}/test/util/easy_time_test.c"
        "${CMAKE_SOURCE_DIR}/test/util/util_test.c"
    )
    target_include_directories(easy_util_test PUBLIC ${EASY_TEST_INC})
    target_link_libraries(easy_util_test ${EASY_TEST_LINK_LIB})

    add_executable(easy_thread_test
        "${CMAKE_SOURCE_DIR}/test/thread/easy_uthread_test.c"
        "${CMAKE_SOURCE_DIR}/test/thread/thread_test.c"
    )
    target_include_directories(easy_thread_test PUBLIC ${EASY_TEST_INC})
    target_link_libraries(easy_thread_test ${EASY_TEST_LINK_LIB})

    add_executable(easy_memory_test
        "${CMAKE_SOURCE_DIR}/test/memory/easy_mem_page_test.c"
        "${CMAKE_SOURCE_DIR}/test/memory/easy_mem_pool_test.c"
        "${CMAKE_SOURCE_DIR}/test/memory/easy_mem_slab_test.c"
        "${CMAKE_SOURCE_DIR}/test/memory/memory_test.c"
    )
    target_include_directories(easy_memory_test PUBLIC ${EASY_TEST_INC})
    target_link_libraries(easy_memory_test ${EASY_TEST_LINK_LIB})

    add_executable(easy_io_test
        "${CMAKE_SOURCE_DIR}/test/io/easy_baseth_pool_test.c"
        "${CMAKE_SOURCE_DIR}/test/io/easy_client_test.c"
        "${CMAKE_SOURCE_DIR}/test/io/easy_connection_test.c"
        "${CMAKE_SOURCE_DIR}/test/io/easy_event_test.c"
        "${CMAKE_SOURCE_DIR}/test/io/easy_file_test.c"
        "${CMAKE_SOURCE_DIR}/test/io/easy_io_test.c"
        "${CMAKE_SOURCE_DIR}/test/io/easy_log_test.c"
        "${CMAKE_SOURCE_DIR}/test/io/easy_request_test.c"
        "${CMAKE_SOURCE_DIR}/test/io/io_test.c"
    )
    target_include_directories(easy_io_test PUBLIC ${EASY_TEST_INC})
    target_link_libraries(easy_io_test ${EASY_TEST_LINK_LIB})

    add_executable(easy_packet_test
    "${CMAKE_SOURCE_DIR}/test/packet/easy_http_handler_test.c"
    "${CMAKE_SOURCE_DIR}/test/packet/easy_kfc_handler_test.c"
    "${CMAKE_SOURCE_DIR}/test/packet/packet_test.c"
    )
    target_include_directories(easy_packet_test PUBLIC ${EASY_TEST_INC})
    target_link_libraries(easy_packet_test ${EASY_TEST_LINK_LIB})
endif() # EASY_BUILD_TEST

#TODO: sample
# TODO: install