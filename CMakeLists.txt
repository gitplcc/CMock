###################################################################################
#                                                                                 #
# NAME: CMakeLists.txt                                                            #
#                                                                                 #
# AUTHOR: Mike Karlesky, Mark VanderVoord, Greg Williams.                         #
# WRITTEN BY: Michael Brockus.                                                    #
#             Luis Castedo                                                        #
#                                                                                 #
# License: MIT                                                                    #
#                                                                                 #
###################################################################################

cmake_minimum_required(VERSION 3.12..3.20)

# Read src/cmock.h file and get project version from it
set(CMOCK_HEADER src/cmock.h)

file(
    STRINGS "${CMOCK_HEADER}" CMOCK_HEADER_CONTENT 
    REGEX "^#define CMOCK_VERSION_(MAJOR|MINOR|BUILD) +[0-9]+$"
)

set(CMOCK_HEADER_VERSION_MAJOR 0)
set(CMOCK_HEADER_VERSION_MINOR 0)
set(CMOCK_HEADER_VERSION_BUILD 0)

foreach(VERSION_LINE IN LISTS CMOCK_HEADER_CONTENT)
    foreach(VERSION_PART MAJOR MINOR BUILD)
        string(
            CONCAT REGEX_STRING "#define CMOCK_VERSION_" 
                                "${VERSION_PART}" 
                                " +([0-9]+)"
        )

        if(VERSION_LINE MATCHES "${REGEX_STRING}")
            set(CMOCK_HEADER_VERSION_${VERSION_PART} "${CMAKE_MATCH_1}")
        endif()
    endforeach()
endforeach()

project(
    CMock
    VERSION     ${CMOCK_HEADER_VERSION_MAJOR}.${CMOCK_HEADER_VERSION_MINOR}.${CMOCK_HEADER_VERSION_BUILD}
    LANGUAGES   C
    DESCRIPTION "C Mocking framework."
)

# Includes ---------------------------------------------------------------------
include(GNUInstallDirs)
include(CMakePackageConfigHelpers)

option(CMOCK_UNITY_INCLUDE_DOUBLE "Build Unity with double support" OFF)
option(CMOCK_UNITY_INCLUDE_PRINT_FORMATTED "Build Unity with formatted print support" OFF)

add_subdirectory(vendor/unity)
target_compile_definitions(unity
    PUBLIC
        $<$<BOOL:${CMOCK_UNITY_INCLUDE_DOUBLE}>:UNITY_INCLUDE_DOUBLE>
        $<$<BOOL:${CMOCK_UNITY_INCLUDE_PRINT_FORMATTED}>:UNITY_INCLUDE_PRINT_FORMATTED>
)

add_subdirectory(src)

write_basic_package_version_file(${PROJECT_NAME}ConfigVersion.cmake
    VERSION       ${PROJECT_VERSION}
    COMPATIBILITY SameMajorVersion 
)

## Target's cmake files: targets export
install(
    EXPORT  ${PROJECT_NAME}Targets
    NAMESPACE   ${PROJECT_NAME}::
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}
)

## Target's cmake files: config and version config for find_package()
install(
  FILES
      ${PROJECT_NAME}Config.cmake
      ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake
  DESTINATION
      ${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}
)
