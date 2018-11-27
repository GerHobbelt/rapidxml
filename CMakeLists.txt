cmake_minimum_required(VERSION 3.5)

# see : https://crascit.com/2016/04/09/using-ccache-with-cmake/
find_program(CCACHE_PROGRAM ccache)
if(CCACHE_PROGRAM)
  set_property(GLOBAL PROPERTY RULE_LAUNCH_COMPILE "${CCACHE_PROGRAM}")
else()
  message("ccache not found, repeated builds will be faster if this program is installed")
endif()

project(rapidxml)

set(PROJECT_VERSION "1.13")

if(NOT CMAKE_INSTALL_PREFIX)
  set(CMAKE_INSTALL_PREFIX /usr/local)
endif()

if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE "Release" CACHE STRING
    "Choose the type of build, options are: Debug Release
      RelWithDebInfo MinSizeRel." FORCE)
endif()

#############################################################################
# Generate library target
#############################################################################
# add_library(rapidxml STATIC src/rapidxml.cpp)
add_library(rapidxml INTERFACE)
get_filename_component(RAPIDXML_INC_DIR ./include ABSOLUTE)
target_include_directories(rapidxml
  # PUBLIC
  INTERFACE
    $<BUILD_INTERFACE:${RAPIDXML_INC_DIR}>
    $<INSTALL_INTERFACE:include>
  )

# add target to export set
install(TARGETS rapidxml
  EXPORT rapidxml-targets
  RUNTIME DESTINATION bin
  LIBRARY DESTINATION lib
  ARCHIVE DESTINATION lib
)

#############################################################################
# Generate the cmake configuration files for the build tree
#############################################################################
set(PROJECT_DESCRIPTION "A fast XML parsing utility.")

include(CMakePackageConfigHelpers)
configure_package_config_file(
  ${PROJECT_SOURCE_DIR}/cmake/Modules/project-config.cmake.in
  ${PROJECT_BINARY_DIR}/rapidxml-config.cmake
  INSTALL_DESTINATION ${PROJECT_BINARY_DIR}
  )

write_basic_package_version_file(
  ${PROJECT_BINARY_DIR}/rapidxml-config-version.cmake
  VERSION ${PROJECT_VERSION}
  COMPATIBILITY SameMajorVersion
  )

# Export project export set to the build tree
export(
  EXPORT rapidxml-targets
  FILE "${PROJECT_BINARY_DIR}/rapidxml-targets.cmake"
  )

# Export the package for use from the build-tree
# (this registers the build-tree with a global CMake-registry ~/.cmake)
export(PACKAGE rapidxml)

#############################################################################
# Generate the cmake configuration files for the install tree
#############################################################################
set(LIB_INSTALL_DIR lib/ CACHE PATH "Library install dir")
set(CMAKE_CONFIG_DEST "${LIB_INSTALL_DIR}/rapidxml/cmake")

configure_package_config_file(
  ${PROJECT_SOURCE_DIR}/cmake/Modules/project-config.cmake.in
  ${PROJECT_BINARY_DIR}/${CMAKE_FILES_DIRECTORY}/${PROJECT_NAME}-config.cmake
  INSTALL_DESTINATION ${CMAKE_CONFIG_DEST}
  )

write_basic_package_version_file(
  ${PROJECT_BINARY_DIR}/${CMAKE_FILES_DIRECTORY}/rapidxml-config-version.cmake
  VERSION ${PROJECT_VERSION}
  COMPATIBILITY SameMajorVersion
  )

# Install cmake config files
install(FILES ${PROJECT_BINARY_DIR}/${CMAKE_FILES_DIRECTORY}/${PROJECT_NAME}-config.cmake
  ${PROJECT_BINARY_DIR}/${CMAKE_FILES_DIRECTORY}/${PROJECT_NAME}-config-version.cmake
  DESTINATION ${CMAKE_CONFIG_DEST})

# Export project export set to the install tree
install(
  EXPORT ${PROJECT_NAME}-targets
  DESTINATION "lib/rapidxml/cmake"
  )

# Install all library headers
install(
  DIRECTORY include/rapidxml
  DESTINATION include
  FILES_MATCHING PATTERN "*.hpp"
  )
