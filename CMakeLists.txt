# cmake file

cmake_minimum_required(VERSION 3.2.0)

set(PRJ_NAME "LinearAllocator")
string(TOUPPER ${PRJ_NAME} PRJ_NAME_UPPER)
string(TOLOWER ${PRJ_NAME} PRJ_NAME_LOWER)

project(${PRJ_NAME} CXX)

enable_testing()

# add project cmake module path
set(PRJ_CMAKE_MODULE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/cmake/modules")
list(APPEND CMAKE_MODULE_PATH ${PRJ_CMAKE_MODULE_DIR})

include(CMakePackageConfigHelpers)
include(Util)

option(PRJ_SKIP_TESTS "Skip configuring the project tests" OFF)
option(PRJ_DEBUG "Enable project custom debug output facilities" ON)

get_version(VERSION PRJ_VERSION)

# CMP0056 is required for any libc++abi dependency of the try_compile() command
# this bumps required version to 3.2
# otherwise LD_LIBRARY_DIR needs to be set
set_policies(CMP0056 NEW)


add_library(${PRJ_NAME} INTERFACE)

target_compile_definitions(${PRJ_NAME} INTERFACE VERSION_STRING=${PRJ_VERSION})

target_include_directories(${PRJ_NAME} INTERFACE
  $<BUILD_INTERFACE:${CMAKE_SOURCE_DIR}/include>)
target_include_directories(${PRJ_NAME} INTERFACE
  $<BUILD_INTERFACE:${CMAKE_BINARY_DIR}/include>)
target_include_directories(${PRJ_NAME} INTERFACE
  $<INSTALL_INTERFACE:${CMAKE_INSTALL_PREFIX}/include>)


attach_compilation_db(TARGET ${PRJ_NAME})


# installation

set(PRJ_INSTALL_INCLUDE_DIR "include")
set(PRJ_INSTALL_SHARE_DIR "share")
set(PRJ_CONFIG_HEADER_FILE "Config.hpp")
set(PRJ_CMAKE_CONFIG_FILE "config.cmake")
set(PRJ_CMAKE_CONFIG_EXTRA_FILE "config-extra.cmake")

install(TARGETS ${PRJ_NAME} EXPORT ${PRJ_NAME} DESTINATION "include")

file(MAKE_DIRECTORY "${CMAKE_CURRENT_BINARY_DIR}/include")
set_directory_properties(PROPERTIES ADDITIONAL_MAKE_CLEAN_FILES
  "${CMAKE_CURRENT_BINARY_DIR}/include")

configure_file(
  "${CMAKE_CURRENT_SOURCE_DIR}/include/${PRJ_CONFIG_HEADER_FILE}.in"
  "${CMAKE_CURRENT_BINARY_DIR}/include/${PRJ_CONFIG_HEADER_FILE}")

configure_file(
  "${CMAKE_CURRENT_SOURCE_DIR}/share/cmake/${PRJ_CMAKE_CONFIG_FILE}.in"
  "${CMAKE_CURRENT_BINARY_DIR}/share/cmake/${PRJ_NAME_LOWER}-${PRJ_CMAKE_CONFIG_FILE}"
  @ONLY)

configure_file(
  "${CMAKE_CURRENT_SOURCE_DIR}/share/cmake/${PRJ_CMAKE_CONFIG_EXTRA_FILE}.in"
  "${CMAKE_CURRENT_BINARY_DIR}/share/cmake/${PRJ_NAME}-${PRJ_CMAKE_CONFIG_EXTRA_FILE}"
  @ONLY)

get_version(SHORT VERSION PRJ_VERSION_SHORT)
write_basic_package_version_file(
  "${CMAKE_CURRENT_BINARY_DIR}/share/cmake/${PRJ_NAME_LOWER}-config-version.cmake"
  VERSION ${PRJ_VERSION_SHORT} COMPATIBILITY SameMajorVersion)

install(DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/include/" 
  DESTINATION ${PRJ_INSTALL_INCLUDE_DIR}
  PATTERN "*.in" EXCLUDE)
install(DIRECTORY "${CMAKE_CURRENT_BINARY_DIR}/include/" 
  DESTINATION ${PRJ_INSTALL_INCLUDE_DIR})
install(DIRECTORY "${CMAKE_CURRENT_BINARY_DIR}/share/"
  DESTINATION ${PRJ_INSTALL_SHARE_DIR})

install(EXPORT ${PRJ_NAME} DESTINATION "${PRJ_INSTALL_SHARE_DIR}/cmake")
