cmake_minimum_required(VERSION 3.12)
project(Tutorial)

include(CheckCSourceCompiles)
include(CheckCXXSourceCompiles)

set(CMAKE_VERBOSE_MAKEFILE ON)
set(CMAKE_CXX_COMPILER clang++)
set(CMAKE_C_COMPILER clang)

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fsanitize=fuzzer")
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fsanitize=fuzzer")

# Add the MyLibrary target
add_library(MyLibrary SHARED my_library.cpp)

# Add the Tutorial executable target
add_executable(Tutorial test_fuzzer.cc)

# Link MyLibrary with Tutorial executable
target_link_libraries(Tutorial PRIVATE MyLibrary)
