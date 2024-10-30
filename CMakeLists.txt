cmake_minimum_required(VERSION 3.2...3.27)

set(PROJECT_NAME ${CMAKE_GIT_PROJECT_PREFIX}cmake_git_version_tracking)
project(${PROJECT_NAME} LANGUAGES C VERSION 1.0)

list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")
# Standard paths
include(install_paths)

# Define the two required variables before including
# the source code for watching a git repository.
set(PRE_CONFIGURE_FILE "git.c.in")
set(POST_CONFIGURE_FILE "${CMAKE_CURRENT_BINARY_DIR}/git.c")
include(git_watcher.cmake)
include(CMakePackageConfigHelpers)

set(CMAKE_CONFIG_FILE "${CMAKE_CURRENT_SOURCE_DIR}/cmake/Config.cmake.in" CACHE INTERNAL "Package config file" )

option(SAVE_GIT_STATE_TO_PY "Creates a .py file containg to be used for python bindings versioning" ON)
if(SAVE_GIT_STATE_TO_PY)
    set(GIT_STATE_PY_PATH ${CMAKE_CURRENT_BINARY_DIR}/${CMAKE_GIT_PROJECT_PREFIX}git_state.py CACHE INTERNAL "Path to py git state")
endif()
# Create a library out of the compiled post-configure file.
# so downstream projects don't suffer from warnings on a
# 3rdparty library.
add_library(${PROJECT_NAME} STATIC ${POST_CONFIGURE_FILE})

target_include_directories(${PROJECT_NAME}
        SYSTEM PUBLIC
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
)

add_dependencies(${PROJECT_NAME} ${CMAKE_GIT_PROJECT_PREFIX}check_git)

# The C99 standard is only required because we're using <stdbool.h>.
# This could be removed if it's a problem for users, but would require the
# cmake configure() commands to translate true/false literals to 1/0.
set_property(TARGET ${PROJECT_NAME} PROPERTY C_STANDARD 99)

set(PROJECT_CONFIG_FILE         "${PROJECT_BINARY_DIR}/${PROJECT_NAME}-config.cmake")
set(PROJECT_CONFIG_VERSION_FILE "${PROJECT_BINARY_DIR}/${PROJECT_NAME}-config-version.cmake")

configure_package_config_file(
        ${CMAKE_CONFIG_FILE}
        ${PROJECT_CONFIG_FILE}
        INSTALL_DESTINATION ${INSTALL_CONFIG_DIR})

# This auto-generates a version file.
write_basic_package_version_file(
        ${PROJECT_CONFIG_VERSION_FILE}
        COMPATIBILITY SameMajorVersion)

install(TARGETS ${PROJECT_NAME}
        EXPORT ${PROJECT_NAME}-targets
        RUNTIME DESTINATION ${INSTALL_RUNTIME_DIR}
        LIBRARY DESTINATION ${INSTALL_LIBRARY_DIR}
        ARCHIVE DESTINATION ${INSTALL_ARCHIVE_DIR})

install(FILES ${HEADER_FILES}
        DESTINATION ${INSTALL_INCLUDE_DIR})


install(FILES
            ${PROJECT_CONFIG_VERSION_FILE}
            ${PROJECT_CONFIG_FILE}
        DESTINATION ${INSTALL_CONFIG_DIR})

install(EXPORT ${PROJECT_NAME}-targets
        FILE ${PROJECT_NAME}-targets.cmake
        NAMESPACE ${PROJECT_NAME}::
        DESTINATION ${INSTALL_CONFIG_DIR})

export(EXPORT ${PROJECT_NAME}-targets
       FILE ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}-targets.cmake
       NAMESPACE ${PROJECT_NAME}::)

export(PACKAGE ${PROJECT_NAME})
