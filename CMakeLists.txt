cmake_minimum_required(VERSION 3.8 FATAL_ERROR)
project(osmpbfreader LANGUAGES CXX)

include(GNUInstallDirs)
set(PACKAGE_NAME osmpbfreader)
set(INSTALL_CONFIGDIR ${CMAKE_INSTALL_LIBDIR}/${PACKAGE_NAME}/cmake)

set(CMAKE_MODULE_PATH "${CMAKE_MODULE_PATH};${CMAKE_SOURCE_DIR}/cmake/modules")


# Library target
find_package(osmpbf REQUIRED)
find_package(Protobuf REQUIRED)
add_library(osmpbfreader INTERFACE)
target_include_directories(osmpbfreader
  INTERFACE
    ${osmpbf_INCLUDE_DIRS}
    ${Protobuf_INCLUDE_DIRS}
    $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}>
    $<INSTALL_INTERFACE:include>
)
target_compile_features(osmpbfreader INTERFACE cxx_std_11)
target_link_libraries(osmpbfreader INTERFACE
  ${Protobuf_LIBRARIES}
  z
  ${osmpbf_LIBRARIES}
)
install(
  TARGETS osmpbfreader
  EXPORT install_targets
)


# Examples
add_executable(example_counter example_counter.cc)
target_link_libraries(example_counter PRIVATE osmpbfreader)
set_target_properties(example_counter PROPERTIES RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/bin")

add_executable(example_routing example_routing.cc)
target_link_libraries(example_routing PRIVATE osmpbfreader)
set_target_properties(example_routing PROPERTIES RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/bin")


# Install headers
install(FILES
  "${PROJECT_SOURCE_DIR}/osmpbfreader.h"
  DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})


# Install targets file
install(EXPORT install_targets
  FILE
    ${PACKAGE_NAME}Targets.cmake
  NAMESPACE
    ${PACKAGE_NAME}::
  DESTINATION
    ${INSTALL_CONFIGDIR}
)


# Install osmpbfreaderConfig.cmake
include(CMakePackageConfigHelpers)
configure_package_config_file(
  ${CMAKE_CURRENT_SOURCE_DIR}/cmake/${PACKAGE_NAME}Config.cmake.in
  ${CMAKE_CURRENT_BINARY_DIR}/${PACKAGE_NAME}Config.cmake
  INSTALL_DESTINATION ${INSTALL_CONFIGDIR}
)
install(FILES
  ${CMAKE_CURRENT_BINARY_DIR}/${PACKAGE_NAME}Config.cmake
  DESTINATION ${INSTALL_CONFIGDIR}
)


# Install find modules
install(DIRECTORY cmake/modules/ DESTINATION ${INSTALL_CONFIGDIR}/modules)
