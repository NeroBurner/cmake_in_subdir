if(${CMAKE_SOURCE_DIR} STREQUAL ${CMAKE_BINARY_DIR})
  message(FATAL_ERROR "Prevented in-tree built. Please create a build directory outside of the project source code and call cmake from there")
endif()

message(STATUS "cmake version ${CMAKE_VERSION}")
cmake_minimum_required(VERSION 3.5)
project(CMakeInSubdir)

include(GNUInstallDirs)

set(CIS_VERSION_MAJOR 0)
set(CIS_VERSION_MINOR 0)
set(CIS_VERSION_PATCH 1)
set(CIS_VERSION "${CIS_VERSION_MAJOR}.${CIS_VERSION_MINOR}.${CIS_VERSION_PATCH}")

configure_file("${CMAKE_CURRENT_SOURCE_DIR}/cis.hpp.in"
  "${CMAKE_CURRENT_BINARY_DIR}/cis.hpp"
  @ONLY)

add_library(CMakeInSubdir INTERFACE)
add_library(CIS::CmakeInSubdir ALIAS CMakeInSubdir)
target_include_directories(CMakeInSubdir INTERFACE
  $<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}>
  $<INSTALL_INTERFACE:include>)

add_executable(cis_version main.cpp)
target_link_libraries(cis_version PRIVATE CMakeInSubdir)

##### Installation targets #####
install(TARGETS CMakeInSubdir EXPORT CISTargets)

##### Export files #####
set(PKG_PREFIX "${CMAKE_INSTALL_LIBDIR}/cmake/CIS")

include(CMakePackageConfigHelpers)
write_basic_package_version_file("${CMAKE_BINARY_DIR}/CISConfigVersion.cmake"
  VERSION ${CIS_VERSION}
  COMPATIBILITY AnyNewerVersion
)

install(EXPORT CISTargets
  FILE CISTargets.cmake
  NAMESPACE CIS::
  DESTINATION ${PKG_PREFIX}
)
install(
  FILES
  ${CMAKE_CURRENT_SOURCE_DIR}/CISConfig.cmake
  ${CMAKE_BINARY_DIR}/CISConfigVersion.cmake
  DESTINATION ${PKG_PREFIX}
)

list(APPEND INCLUDE_FILES ${BIN_INCLUDE_FILES})
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/cis.hpp DESTINATION include)

