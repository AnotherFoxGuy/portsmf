cmake_minimum_required(VERSION 3.1)

project(PortSMF VERSION 235.1 LANGUAGES CXX)
set(CMAKE_CXX_STANDARD 11)

if(CMAKE_CXX_COMPILER_ID MATCHES "Clang"
   OR CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
  add_compile_options(-Wall -Wextra -pedantic)
elseif(CMAKE_CXX_COMPILER_ID STREQUAL "MSVC")
  add_compile_options(/W4)
endif()

option(BUILD_SHARED_LIBS "Build dynamic library" ON)

add_library(PortSMF
  src/allegro.cpp
  src/allegrord.cpp
  src/allegroserial.cpp
  src/allegrosmfrd.cpp
  src/allegrosmfwr.cpp
  src/allegrowr.cpp
  src/mfmidi.cpp
  src/strparse.cpp
)
target_include_directories(PortSMF PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
  $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
)
set_target_properties(PortSMF PROPERTIES OUTPUT_NAME portSMF)
set_target_properties(PortSMF PROPERTIES
  WINDOWS_EXPORT_ALL_SYMBOLS TRUE
)

include(GNUInstallDirs)
include(CMakePackageConfigHelpers)

# Library
install(TARGETS PortSMF
  EXPORT PortSMFTargets
  ARCHIVE DESTINATION "${CMAKE_INSTALL_LIBDIR}"
  LIBRARY DESTINATION "${CMAKE_INSTALL_LIBDIR}"
  RUNTIME DESTINATION "${CMAKE_INSTALL_BINDIR}"
  INCLUDES DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}"
)

# Headers
install(FILES
  include/allegro.h
  include/mfmidi.h
  include/strparse.h
  include/trace.h
  DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}/portSMF")

# pkgconfig
configure_file(${CMAKE_CURRENT_SOURCE_DIR}/packaging/portSMF.pc.in
  ${CMAKE_CURRENT_BINARY_DIR}/packaging/portSMF.pc @ONLY)
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/packaging/portSMF.pc DESTINATION "${CMAKE_INSTALL_LIBDIR}/pkgconfig")

# CMake config
set(PORTSMF_INSTALL_CMAKEDIR "lib/cmake/PortSMF")
install(
  EXPORT PortSMFTargets
  FILE PortSMFTargets.cmake
  NAMESPACE PortSMF::
  DESTINATION "${PORTSMF_INSTALL_CMAKEDIR}"
)
configure_package_config_file(packaging/PortSMFConfig.cmake.in
  "${CMAKE_CURRENT_BINARY_DIR}/packaging/PortSMFConfig.cmake"
  INSTALL_DESTINATION "${PORTSMF_INSTALL_CMAKEDIR}"
)
write_basic_package_version_file(
  "${CMAKE_CURRENT_BINARY_DIR}/packaging/PortSMFConfigVersion.cmake"
  VERSION "${CMAKE_PROJECT_VERSION}"
  COMPATIBILITY SameMajorVersion
)
install(
  FILES
    "${CMAKE_CURRENT_BINARY_DIR}/packaging/PortSMFConfig.cmake"
    "${CMAKE_CURRENT_BINARY_DIR}/packaging/PortSMFConfigVersion.cmake"
  DESTINATION "${PORTSMF_INSTALL_CMAKEDIR}"
)

option(BUILD_APPS "Build example applications" OFF)
if(BUILD_APPS)
  add_subdirectory(apps)
endif()

option(BUILD_TESTING "Include test projects" OFF)
if(BUILD_TESTING)
  add_subdirectory(test)
endif()
