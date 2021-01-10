cmake_minimum_required(VERSION 3.0)
project(sqlite3 C)

set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
set(CMAKE_DISABLE_IN_SOURCE_BUILD ON)
set(CMAKE_DISABLE_SOURCE_CHANGES  ON)

if (TARGET sqlite3)
    return()
endif(TARGET sqlite3)

add_subdirectory(src)
# vi:ai et sw=4 ts=4:
