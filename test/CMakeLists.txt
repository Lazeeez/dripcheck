cmake_minimum_required(VERSION 3.12)
project(Tutorial)

include(CheckCSourceCompiles)
include(CheckCXXSourceCompiles)
# We need some extra FAIL_REGEX patterns
# Note that CHECK_C_SOURCE_COMPILES is a misnomer, it will also link.
SET(fail_patterns
    FAIL_REGEX "argument unused during compilation"
    FAIL_REGEX "unsupported .*option"
    FAIL_REGEX "unknown .*option"
    FAIL_REGEX "unrecognized .*option"
    FAIL_REGEX "ignoring unknown option"
    FAIL_REGEX "warning:.*ignored"
    FAIL_REGEX "warning:.*is valid for.*but not for"
    FAIL_REGEX "warning:.*redefined"
    FAIL_REGEX "[Ww]arning: [Oo]ption"
    )

MACRO (MY_CHECK_C_COMPILER_FLAG flag)
  STRING(REGEX REPLACE "[-,= +]" "_" result "have_C_${flag}")
  SET(SAVE_CMAKE_REQUIRED_FLAGS "${CMAKE_REQUIRED_FLAGS}")
  SET(CMAKE_REQUIRED_FLAGS "${CMAKE_REQUIRED_FLAGS} ${flag}")
  CHECK_C_SOURCE_COMPILES("
    #include <iostream>
    extern \"C\" int LLVMFuzzerTestOneInput(const uint8_t *data, size_t size) {
      return 0;
    }
  " ${result} ${fail_patterns})
  SET(CMAKE_REQUIRED_FLAGS "${SAVE_CMAKE_REQUIRED_FLAGS}")
ENDMACRO()


MACRO (MY_CHECK_CXX_COMPILER_FLAG flag)
  STRING(REGEX REPLACE "[-,= +]" "_" result "have_CXX_${flag}")
  SET(SAVE_CMAKE_REQUIRED_FLAGS "${CMAKE_REQUIRED_FLAGS}")
  SET(CMAKE_REQUIRED_FLAGS "${CMAKE_REQUIRED_FLAGS} ${flag}")
  CHECK_CXX_SOURCE_COMPILES("
    #include <iostream>
    extern \"C\" int LLVMFuzzerTestOneInput(const uint8_t *data, size_t size) {
      return 0;
    }
  " ${result} ${fail_patterns})
  SET(CMAKE_REQUIRED_FLAGS "${SAVE_CMAKE_REQUIRED_FLAGS}")
ENDMACRO()

FUNCTION(MY_CHECK_AND_SET_COMPILER_FLAG flag)
  # At the moment this is gcc-only.
  # Let's avoid expensive compiler tests on Windows:
  IF(WIN32)
    RETURN()
  ENDIF()
  MY_CHECK_C_COMPILER_FLAG(${flag})
  MY_CHECK_CXX_COMPILER_FLAG(${flag})
  STRING(REGEX REPLACE "[-,= +]" "_" result "${flag}")
  FOREACH(lang C CXX)
    IF (have_${lang}_${result})
      IF(ARGN)
        FOREACH(type ${ARGN})
          SET(CMAKE_${lang}_FLAGS_${type} "${CMAKE_${lang}_FLAGS_${type}} ${flag}" PARENT_SCOPE)
        ENDFOREACH()
      ELSE()
        SET(CMAKE_${lang}_FLAGS "${CMAKE_${lang}_FLAGS} ${flag}" PARENT_SCOPE)
      ENDIF()
    ENDIF()
  ENDFOREACH()
ENDFUNCTION()

set(CMAKE_VERBOSE_MAKEFILE ON)

# set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fsanitize=fuzzer")
# set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fsanitize=fuzzer")
MY_CHECK_AND_SET_COMPILER_FLAG("-fsanitize=fuzzer")

# Add the MyLibrary target
add_library(MyLibrary SHARED my_library.cpp)

# Add the Tutorial executable target
add_executable(Tutorial test_fuzzer.cc)

# Link MyLibrary with Tutorial executable
target_link_libraries(Tutorial PRIVATE MyLibrary)
