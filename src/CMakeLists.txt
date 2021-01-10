set(CMAKE_POSITION_INDEPENDENT_CODE ON)

if(${CMAKE_SYSTEM_NAME} MATCHES "Darwin")
    set(CMAKE_SHARED_MODULE_CREATE_C_FLAGS
        "${CMAKE_SHARED_MODULE_CREATE_C_FLAGS} -undefined dynamic_lookup")
    cmake_policy(SET CMP0042 NEW)
endif()

# message("Current binary dir: ${CMAKE_CURRENT_BINARY_DIR} source dir: ${CMAKE_CURRENT_SOURCE_DIR}")
# Current binary dir: /Users/matt/repos/base/kvidxkit/build/deps/sqlite3/src
# Current source dir: /Users/matt/repos/base/kvidxkit/deps/sqlite3/src

# If you need debug, build with:
# cmake -DCMAKE_BUILD_TYPE=Debug ..
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wall -Wextra -std=gnu11")
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wno-missing-field-initializers")
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wno-unused-parameter")
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wno-unused-function")
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wstrict-aliasing -Wstrict-overflow")
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wno-sign-compare -Wno-unused-const-variable")

if(CMAKE_BUILD_TYPE MATCHES Debug)
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -O0")
    add_definitions(-DSQLITE_ENABLE_EXPLAIN_COMMENTS)
else()
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -O3")
endif()

set(SQLITE_WIN_BUILD OFF)

if(${CMAKE_SYSTEM_NAME} MATCHES "Windows")
    set(SQLITE_WIN_BUILD ON)
endif()

# Configuration
# All options are defined at:
# https://www.sqlite.org/compile.html
add_definitions(
    -DSQLITE_DEFAULT_FILE_FORMAT=4
    -DSQLITE_DEFAULT_FILE_PERMISSIONS=0600
    -DSQLITE_MAX_PAGE_COUNT=2147483646 # DB size 128 TiB with 65536 page size
    -DSQLITE_DEFAULT_PAGE_SIZE=8192    # without overwrite, default is 4096
    -DSQLITE_DEFAULT_CACHE_SIZE=-8000  # 8 MB page cache
    -DSQLITE_DEFAULT_FOREIGN_KEYS=1       # enable foreign keys without pragmas
    -DSQLITE_DEFAULT_RECURSIVE_TRIGGERS=1 # allow recursive foreign key triggers
    -DSQLITE_MAX_TRIGGER_DEPTH=1073741824 # depth of recursive cleanup ops
    -DHAVE_MALLOC_USABLE_SIZE
    -DHAVE_USLEEP
    -DSQLITE_DIRECT_OVERFLOW_READ

    # These defines make all opcodes exist so our dispatch
    # table can resolve all goto labels
    -DSQLITE_ENABLE_OFFSET_SQL_FUNC
    -DSQLITE_ENABLE_COLUMN_USED_MASK
    -DSQLITE_ENABLE_CURSOR_HINTS

    # Things we don't want included in our build
    # Disable UTF16 string functions
    -DSQLITE_OMIT_UTF16
    -DSQLITE_OMIT_DECLTYPE
    -DSQLITE_OMIT_DEPRECATED
    -DSQLITE_OMIT_PROGRESS_CALLBACK
    # -DSQLITE_OMIT_INCRBLOB # <-- RTREE NEEDS THIS OMIT OMITTED
    #    -DSQLITE_UNTESTABLE
    #    -DSQLITE_DEFAULT_AUTOVACUUM=2
    #    -DSQLITE_THREADSAFE=2
    #    Optional: if threading not needed, disable concurrency


    # Enable overwrite-deletes-with-zeroes
    # -DSQLITE_SECURE_DELETE

    # Enable bulk copy ability natively built into sqlite
    # https://www.sqlite.org/c3ref/serialize.html
    -DSQLITE_ENABLE_DESERIALIZE

    # Enable FTS
    # https://www.sqlite.org/fts5.html
    -DSQLITE_ENABLE_FTS5

    # Enable the R*Tree
    # https://www.sqlite.org/rtree.html
    # https://www.sqlite.org/geopoly.html
    -DSQLITE_ENABLE_RTREE
    -DSQLITE_ENABLE_GEOPOLY

    # Enable JSON SQL functions
    # https://www.sqlite.org/json1.html
    -DSQLITE_ENABLE_JSON1

    # Allow packing changes into a "changeset" we can ship to other servers
    # https://www.sqlite.org/sessionintro.html
    -DSQLITE_ENABLE_SESSION
    -DSQLITE_ENABLE_PREUPDATE_HOOK

    # Enable "Resumable Bulk Update" capability for also send/receive changes
    # https://www.sqlite.org/rbu.html
    -DSQLITE_ENABLE_RBU

    # Omit NULL columns at the end of rows entirely
    -DSQLITE_ENABLE_NULL_TRIM
)

if(${CMAKE_SYSTEM_NAME} MATCHES "Linux")
    add_definitions(-DHAVE_FDATASYNC)
endif()

add_library(sqlite3 OBJECT sqlite3.c)

add_library(sqlite3-static STATIC $<TARGET_OBJECTS:sqlite3>)

if(NOT APPLE)
    target_link_libraries(sqlite3-static pthread)
    target_link_libraries(sqlite3-static dl)
endif()


if (SQLITE_WIN_BUILD)
    add_definitions(
        -DSQLITE_4_BYTE_ALIGNED_MALLOC
    )
    set_target_properties(sqlite3-shared PROPERTIES DEFINE_SYMBOL
                                "SQLITE_API=__declspec(dllexport)")
endif()

if(Build32Bit)
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -m32")
    set_target_properties(sqlite3-shared
                          PROPERTIES COMPILE_FLAGS "-m32" LINK_FLAGS "-m32")
    set_target_properties(sqlite3-static
                          PROPERTIES COMPILE_FLAGS "-m32" LINK_FLAGS "-m32")
endif()
