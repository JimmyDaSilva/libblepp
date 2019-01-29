cmake_minimum_required(VERSION 3.4)
project(blepp)

set(FOOBAR_MAJOR_VERSION 0)
set(FOOBAR_MINOR_VERSION 5)
set(FOOBAR_PATCH_VERSION 0)
set(FOOBAR_VERSION
  ${FOOBAR_MAJOR_VERSION}.${FOOBAR_MINOR_VERSION}.${FOOBAR_PATCH_VERSION})

# Offer the user the choice of overriding the installation directories
set(INSTALL_LIB_DIR lib CACHE PATH "Installation directory for libraries")
set(INSTALL_BIN_DIR bin CACHE PATH "Installation directory for executables")
set(INSTALL_INCLUDE_DIR include CACHE PATH
  "Installation directory for header files")
if(WIN32 AND NOT CYGWIN)
  set(DEF_INSTALL_CMAKE_DIR cmake)
else()
  set(DEF_INSTALL_CMAKE_DIR lib/cmake/blepp)
endif()
set(INSTALL_CMAKE_DIR ${DEF_INSTALL_CMAKE_DIR} CACHE PATH
  "Installation directory for CMake files")


set(CMAKE_CXX_STANDARD 11)

set(HEADERS
    blepp/bledevice.h
    blepp/logging.h
    blepp/float.h
    blepp/uuid.h
    blepp/pretty_printers.h
    blepp/gap.h
    blepp/lescan.h
    blepp/xtoa.h
    blepp/att.h
    blepp/blestatemachine.h
    blepp/att_pdu.h)

set(SRC
    src/att_pdu.cc
    src/float.cc
    src/logging.cc
    src/uuid.cc
    src/blestatemachine.cc
    src/bledevice.cc
    src/pretty_printers.cc
    src/att.cc
    src/lescan.cc
    ${HEADERS})

set(EXAMPLES
    examples/lescan.cc
    examples/blelogger.cc
    examples/bluetooth.cc
    examples/lescan_simple.cc
    examples/temperature.cc)

set(CMAKE_MODULE_PATH ${CMAKE_SOURCE_DIR}/cmake/modules)

find_package(Bluez REQUIRED)

include_directories(${PROJECT_SOURCE_DIR} ${BLUEZ_INCLUDE_DIRS})
add_library(${PROJECT_NAME} SHARED ${SRC})
set_target_properties(${PROJECT_NAME} PROPERTIES SOVERSION 5)

foreach (example_src ${EXAMPLES})
    get_filename_component(example_name ${example_src} NAME_WE)

    add_executable(${example_name} ${example_src})
    target_link_libraries(${example_name} ${PROJECT_NAME} ${BLUEZ_LIBRARIES})
    set_target_properties(${example_name} PROPERTIES RUNTIME_OUTPUT_DIRECTORY examples)
endforeach()

install(TARGETS ${PROJECT_NAME} LIBRARY DESTINATION lib)
install(DIRECTORY blepp DESTINATION include)


# Make relative paths absolute (needed later on)
foreach(p LIB BIN INCLUDE CMAKE)
  set(var INSTALL_${p}_DIR)
  if(NOT IS_ABSOLUTE "${${var}}")
    set(${var} "${CMAKE_INSTALL_PREFIX}/${${var}}")
  endif()
endforeach()

# The interesting stuff goes here
# ===============================

# Add all targets to the build-tree export set
#export(TARGETS 
#  FILE "${PROJECT_BINARY_DIR}/BLEPPTargets.cmake")

# Export the package for use from the build-tree
# (this registers the build-tree with a global CMake-registry)
export(PACKAGE blepp)

# Create the FooBarConfig.cmake and FooBarConfigVersion files
file(RELATIVE_PATH REL_INCLUDE_DIR "${INSTALL_CMAKE_DIR}"
   "${INSTALL_INCLUDE_DIR}")
# ... for the build tree
set(CONF_INCLUDE_DIRS "${PROJECT_SOURCE_DIR}" "${PROJECT_BINARY_DIR}")
configure_file(bleppConfig.cmake.in
  "${PROJECT_BINARY_DIR}/bleppConfig.cmake" @ONLY)
# ... for the install tree
set(CONF_INCLUDE_DIRS "\${BLEPP_CMAKE_DIR}/${REL_INCLUDE_DIR}")
configure_file(bleppConfig.cmake.in
  "${PROJECT_BINARY_DIR}${CMAKE_FILES_DIRECTORY}/bleppConfig.cmake" @ONLY)
# ... for both
configure_file(bleppConfigVersion.cmake.in
  "${PROJECT_BINARY_DIR}/bleppConfigVersion.cmake" @ONLY)

# Install the bleppConfig.cmake and bleppConfigVersion.cmake
install(FILES
  "${PROJECT_BINARY_DIR}${CMAKE_FILES_DIRECTORY}/bleppConfig.cmake"
  "${PROJECT_BINARY_DIR}/bleppConfigVersion.cmake"
  DESTINATION "${INSTALL_CMAKE_DIR}" COMPONENT dev)

# Install the export set for use with the install-tree
#install(EXPORT BLEPPTargets DESTINATION
#  "${INSTALL_CMAKE_DIR}" COMPONENT dev)

