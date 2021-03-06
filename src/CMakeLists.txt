cmake_minimum_required(VERSION 3.1.0)
project(HowToWriteAHeapMemoryProfiler)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED On)
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${CMAKE_CURRENT_SOURCE_DIR}/cmake)

find_package(Libunwind REQUIRED)
find_package(LibElf REQUIRED)
find_package(Elfutils REQUIRED)
include(CheckCXXSourceCompiles)

check_cxx_source_compiles(
    "#include <stdio_ext.h>
    #include <fcntl.h>
    #include <dlfcn.h>
    #include <link.h>
    int main() { return 0; }"
    HAVE_LINUX_HEADERS)

if (NOT HAVE_LINUX_HEADERS)
    message(FATAL_ERROR "You are missing some Linux headers required to compile heaptrack.")
endif()

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wpedantic")

add_subdirectory(test_clients)

add_subdirectory(preload)

include_directories(shared)
add_subdirectory(preload_more)

include_directories(${LIBUNWIND_INCLUDE_DIR})
add_subdirectory(backtrace)
add_subdirectory(mappings)

include_directories(
    ${LIBELF_INCLUDE_DIRS}
    ${LIBDW_INCLUDE_DIR}/elfutils
    ${LIBDWARF_INCLUDE_DIRS}
)
add_subdirectory(symbolization)
add_subdirectory(symbolization_inlines)
add_subdirectory(symbolization_clang)

add_subdirectory(attach)
add_subdirectory(got_overwriting)
add_subdirectory(attach_intercept)

add_subdirectory(interning)
