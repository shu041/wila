cmake_minimum_required(VERSION 2.8)

IF(DEFINED CMAKE_BUILD_TYPE)
    SET(CMAKE_BUILD_TYPE ${CMAKE_BUILD_TYPE} CACHE STRING "Choose the type of
build, options are: None(CMAKE_CXX_FLAGS or CMAKE_C_FLAGS used) Coverage Debug
Release RelWithDebInfo MinSizeRel.")
ELSE()
    SET(CMAKE_BUILD_TYPE Debug CACHE STRING "Choose the type of build,
options are: None(CMAKE_CXX_FLAGS or CMAKE_C_FLAGS used) Coverage Debug Release
RelWithDebInfo MinSizeRel.")
ENDIF()

SET (VERSION 0.5.8)

## start setting
SET (this_target wila)
PROJECT(${this_target})

## section: source files
# Add your source files here (one file per line), please SORT in alphabetical order for future maintenance
SET (${this_target}_SOURCE_FILES
    src/sce.cpp
)

## section: header files
# Add your header files here(one file per line), please SORT in alphabetical order for future maintenance!
SET(${this_target}_HEADER_FILES
    include/wila/core.hpp
    include/wila/evaluations.hpp
    include/wila/random.hpp
    include/wila/sce.hpp
    include/wila/sce.h
    include/wila/utils.hpp
)

# Use the FindBoost macro to find boost.
# Note that if boost is installed in a non-standard location you may
# need to help it by setting some variables. For example, on
# CENTOS 5 or 6:
#     cmake -DBOOST_INCLUDEDIR=/usr/include/boost148 \
#           -DBOOST_LIBRARYDIR=/usr/lib64/boost148 .
find_package(Boost 1.48.0 COMPONENTS system date_time thread REQUIRED)

IF(DEFINED ENV{INTEL_CC_HOME})
  message("Found env var INTEL_CC_HOME, using TBB_ROOT_DIR=$ENV{INTEL_CC_HOME}/tbb as a hint for TBB package discovery")
  set(TBB_ROOT_DIR $ENV{INTEL_CC_HOME}/tbb)
ENDIF()

list(APPEND CMAKE_MODULE_PATH "${PROJECT_SOURCE_DIR}")

if (CMAKE_BUILD_TYPE STREQUAL "Coverage")
    list(APPEND CMAKE_MODULE_PATH "${PROJECT_SOURCE_DIR}/../../cmake_modules")
    INCLUDE(CodeCoverage)
endif()

find_package(TBB REQUIRED)
#find_path(TBB_INCLUDE_DIRS concurrent_vector.h PATH_SUFFIXES tbb)
#message ("The path to the TBB headers is ${TBB_INCLUDE_DIRS}")
#find_library (TBB_LIBRARIES NAMES tbb)
message ("The path to the Thread Building Blocks (tbb) library is ${TBB_LIBRARIES}")

find_path(THREADPOOL_INCLUDE_DIRS boost/threadpool.hpp PATHS threadpool ../threadpool)

INCLUDE_DIRECTORIES(
    ./include
    ./include/wila
    ../include
    ${Boost_INCLUDE_DIRS}
    ${TBB_INCLUDE_DIRS}
    ${THREADPOOL_INCLUDE_DIRS}
)

## section: precompiled header
SET_SOURCE_FILES_PROPERTIES(${this_target}_HEADER_FILES
                            PROPERTIES HEADER_FILE_ONLY TRUE)
LIST(APPEND ${this_target}_SOURCE_FILES ${${this_target}_HEADER_FILES})

# JM needed to use this as of 2015-02, after an upgrade to latest Debian setup. The Add_definitions macro was not working, somehow.
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")

## section: add target
ADD_LIBRARY(${this_target} SHARED ${${this_target}_SOURCE_FILES} )
ADD_EXECUTABLE(wila_tests tests/common.cpp tests/main.cpp)
TARGET_LINK_LIBRARIES(wila_tests
    pthread
    ${this_target}
    ${Boost_LIBRARIES}
    ${TBB_LIBRARIES}
)

# Files to be installed - used by packaging stuff below
INSTALL(FILES ${${this_target}_HEADER_FILES} DESTINATION include/wila)
INSTALL(TARGETS ${this_target}
  LIBRARY DESTINATION lib
)
INSTALL(TARGETS wila_tests DESTINATION bin)

# Packages - currently just an RPM
set(CPACK_PACKAGE_VERSION ${VERSION})
set(CPACK_GENERATOR "RPM;TGZ")
set(CPACK_PACKAGE_NAME "wila")
set(CPACK_RPM_PACKAGE_RELEASE 1)
set(CPACK_PACKAGE_RELEASE ${CPACK_RPM_PACKAGE_RELEASE})
set(CPACK_PACKAGE_CONTACT "J-M <jean-michel.perraud@csiro.au>")
set(CPACK_PACKAGE_VENDOR "https://github.com/jmp75/wila")
set(CPACK_PACKAGING_INSTALL_PREFIX ${CMAKE_INSTALL_PREFIX})
set(CPACK_PACKAGE_FILE_NAME "${CPACK_PACKAGE_NAME}-${CPACK_PACKAGE_VERSION}-${CPACK_PACKAGE_RELEASE}.${CMAKE_SYSTEM_PROCESSOR}")
include(CPack)

# Add new build types
message("* Adding build types...")
SET(CMAKE_CXX_FLAGS_COVERAGE
    "${GCC_DEBUG_FLAGS} -g -O0 -fprofile-arcs -ftest-coverage"
    CACHE STRING "Flags used by the C++ compiler during coverage builds."
    FORCE )
SET(CMAKE_C_FLAGS_COVERAGE
    "${GCC_DEBUG_FLAGS} -g -O0 -fprofile-arcs -ftest-coverage"
    CACHE STRING "Flags used by the C compiler during coverage builds."
    FORCE )
SET(CMAKE_EXE_LINKER_FLAGS_COVERAGE
    ""
    CACHE STRING "Flags used for linking binaries during coverage builds."
    FORCE )
SET(CMAKE_SHARED_LINKER_FLAGS_COVERAGE
    ""
    CACHE STRING "Flags used by the shared libraries linker during coverage builds."
    FORCE )
MARK_AS_ADVANCED(
    CMAKE_CXX_FLAGS_COVERAGE
    CMAKE_C_FLAGS_COVERAGE
    CMAKE_EXE_LINKER_FLAGS_COVERAGE
    CMAKE_SHARED_LINKER_FLAGS_COVERAGE )
