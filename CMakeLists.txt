cmake_minimum_required(VERSION 3.1...3.14)

##
## PROJECT
## name and version
##
project(samiralavi_activecpp VERSION 0.1.0 DESCRIPTION "Active Objects for Modern C++" LANGUAGES CXX)


##
## MAIN_PROJECT CHECK
## determine if activecpp is built as a subproject (using add_subdirectory) or if it is the main project
##
set(MAIN_PROJECT OFF)
if (CMAKE_CURRENT_SOURCE_DIR STREQUAL CMAKE_SOURCE_DIR)
    set(MAIN_PROJECT ON)
endif()

##
## OPTIONS
##

if (POLICY CMP0077)
    # Allow CMake 3.13+ to override options when using FetchContent / add_subdirectory.
    cmake_policy(SET CMP0077 NEW)
endif ()

# VERSION_GREATER_EQUAL is not available in CMake 3.1
if(${MAIN_PROJECT} AND (${CMAKE_VERSION} VERSION_EQUAL 3.13 OR ${CMAKE_VERSION} VERSION_GREATER 3.13))
    set(ACTIVECPP_BuildTests_INIT ON)
else()
    set(ACTIVECPP_BuildTests_INIT OFF)
endif()

option(ACTIVECPP_BuildTests                     "Build the unit tests when BUILD_TESTING is enabled." ${ACTIVECPP_BuildTests_INIT})

##
## CONFIGURATION
##
include(GNUInstallDirs)

set(SAMIRALAVI_ACTIVECPP_TARGET_NAME                 ${PROJECT_NAME})
set(SAMIRALAVI_ACTIVECPP_CONFIG_INSTALL_DIR         "${CMAKE_INSTALL_DATADIR}/cmake/${PROJECT_NAME}" CACHE INTERNAL "")
set(SAMIRALAVI_ACTIVECPP_INCLUDE_INSTALL_DIR        "${CMAKE_INSTALL_INCLUDEDIR}")
set(SAMIRALAVI_ACTIVECPP_TARGETS_EXPORT_NAME        "${PROJECT_NAME}Targets")
set(SAMIRALAVI_ACTIVECPP_CMAKE_CONFIG_TEMPLATE      "cmake/config.cmake.in")
set(SAMIRALAVI_ACTIVECPP_CMAKE_CONFIG_DIR           "${CMAKE_CURRENT_BINARY_DIR}")
set(SAMIRALAVI_ACTIVECPP_CMAKE_VERSION_CONFIG_FILE  "${SAMIRALAVI_ACTIVECPP_CMAKE_CONFIG_DIR}/${PROJECT_NAME}ConfigVersion.cmake")
set(SAMIRALAVI_ACTIVECPP_CMAKE_PROJECT_CONFIG_FILE  "${SAMIRALAVI_ACTIVECPP_CMAKE_CONFIG_DIR}/${PROJECT_NAME}Config.cmake")
set(SAMIRALAVI_ACTIVECPP_CMAKE_PROJECT_TARGETS_FILE  "${SAMIRALAVI_ACTIVECPP_CMAKE_CONFIG_DIR}/${PROJECT_NAME}Targets.cmake")
set(SAMIRALAVI_ACTIVECPP_PKGCONFIG_INSTALL_DIR      "${CMAKE_INSTALL_DATADIR}/pkgconfig")
set(SAMIRALAVI_ACTIVECPP_INCLUDE_BUILD_DIR          "${PROJECT_SOURCE_DIR}/include/")

##
## TARGET
## create target and add include path
##
add_library(${SAMIRALAVI_ACTIVECPP_TARGET_NAME} INTERFACE)
add_library(${PROJECT_NAME}::${SAMIRALAVI_ACTIVECPP_TARGET_NAME} ALIAS ${SAMIRALAVI_ACTIVECPP_TARGET_NAME})
target_include_directories(${SAMIRALAVI_ACTIVECPP_TARGET_NAME} INTERFACE include)

# Install a pkg-config file, so other tools can find this.
CONFIGURE_FILE(
    "${CMAKE_CURRENT_SOURCE_DIR}/cmake/pkg-config.pc.in"
    "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}.pc"
)

##
## TESTS
## create and configure the unit test target
##
if (ACTIVECPP_BuildTests)
    include(CTest)
    enable_testing()
endif()

##
## INSTALL
## install header files, generate and install cmake config files for find_package()
##
include(CMakePackageConfigHelpers)
# use a custom package version config file instead of
# write_basic_package_version_file to ensure that it's architecture-independent
configure_file(
    "cmake/samiralavi_activecppConfigVersion.cmake.in"
    ${SAMIRALAVI_ACTIVECPP_CMAKE_VERSION_CONFIG_FILE}
    @ONLY
)
configure_file(
    ${SAMIRALAVI_ACTIVECPP_CMAKE_CONFIG_TEMPLATE}
    ${SAMIRALAVI_ACTIVECPP_CMAKE_PROJECT_CONFIG_FILE}
    @ONLY
)

if(ACTIVECPP_Install)
    install(
        DIRECTORY ${SAMIRALAVI_ACTIVECPP_INCLUDE_BUILD_DIR}
        DESTINATION ${SAMIRALAVI_ACTIVECPP_INCLUDE_INSTALL_DIR}
    )
    install(
        FILES ${SAMIRALAVI_ACTIVECPP_CMAKE_PROJECT_CONFIG_FILE} ${SAMIRALAVI_ACTIVECPP_CMAKE_VERSION_CONFIG_FILE}
        DESTINATION ${SAMIRALAVI_ACTIVECPP_CONFIG_INSTALL_DIR}
    )
    export(
        TARGETS ${SAMIRALAVI_ACTIVECPP_TARGET_NAME}
        NAMESPACE ${PROJECT_NAME}::
        FILE ${SAMIRALAVI_ACTIVECPP_CMAKE_PROJECT_TARGETS_FILE}
    )
    install(
        TARGETS ${SAMIRALAVI_ACTIVECPP_TARGET_NAME}
        EXPORT ${SAMIRALAVI_ACTIVECPP_TARGETS_EXPORT_NAME}
        INCLUDES DESTINATION ${SAMIRALAVI_ACTIVECPP_INCLUDE_INSTALL_DIR}
    )
    install(
        EXPORT ${SAMIRALAVI_ACTIVECPP_TARGETS_EXPORT_NAME}
        NAMESPACE ${PROJECT_NAME}::
        DESTINATION ${SAMIRALAVI_ACTIVECPP_CONFIG_INSTALL_DIR}
    )
    install(
        FILES "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}.pc"
        DESTINATION ${SAMIRALAVI_ACTIVECPP_PKGCONFIG_INSTALL_DIR}
    )
endif()
