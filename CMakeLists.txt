project(qmenumodel)

cmake_minimum_required(VERSION 2.8.9)

# Standard install paths
include(GNUInstallDirs)

find_package(Qt5Core REQUIRED)
find_package(Qt5Qml REQUIRED)
find_package(Qt5Gui REQUIRED)
include(FindPkgConfig)
pkg_check_modules(GLIB REQUIRED glib-2.0>=2.32)
pkg_check_modules(GIO REQUIRED gio-2.0>=2.32)
set(CMAKE_AUTOMOC ON)
set(CMAKE_INCLUDE_CURRENT_DIR ON)
SET(CMAKE_CXX_FLAGS "-std=c++11")
add_definitions(-DQT_NO_KEYWORDS)

find_program(DBUS_RUNNER dbus-test-runner)

# Cooverage tools
OPTION(ENABLE_COVERAGE "Build with coverage analysis support" OFF)
if(ENABLE_COVERAGE)
    message(STATUS "Using coverage flags")
    find_program(COVERAGE_COMMAND gcov)
    if(NOT COVERAGE_COMMAND)
        message(FATAL_ERROR "gcov command not found")
    endif()
    SET(CMAKE_C_FLAGS "-g -O0 -Wall --coverage")
    SET(CMAKE_CXX_FLAGS "-g -O0 -Wall -std=c++11 --coverage")
    SET(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} --coverage")
    include(${CMAKE_SOURCE_DIR}/cmake/lcov.cmake)
endif()

add_subdirectory(libqmenumodel)

# Tests Tools
OPTION(TEST_XML_OUTPUT "Print test results on xml files" ON)
if(NOT DBUS_RUNNER)
    message(STATUS "dbus-test-runner not found tests disabled.")
else()
    # We need to enable test to create the 'make check' target mandatory on jenkins
    enable_testing()
    if(CMAKE_SYSTEM_PROCESSOR STREQUAL "armv7l")
        message(STATUS "Current version of qemu crashes during the tests. We will skip it for now.")
    else()
        message(STATUS "Tests enabled for arch: ${CMAKE_SYSTEM_PROCESSOR}")
        add_subdirectory(tests)
    endif()
endif()

# Doc
OPTION(GENERATE_DOC "Enable qdoc generation" OFF)
if(GENERATE_DOC)
    message(STATUS "QDoc enabled.")
    find_program(QDOC_BIN qdoc)
    if(NOT QDOC_BIN)
        message(FATAL_ERROR "qdoc command not found")
    else()
        add_subdirectory(doc)
    endif()
endif()

