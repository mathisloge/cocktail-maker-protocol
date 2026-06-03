# SPDX-FileCopyrightText: 2025 Mathis Logemann <mathis@quite.rocks>
#
# SPDX-License-Identifier: GPL-3.0-or-later

cmake_minimum_required(VERSION 4.3)
file(READ "${CMAKE_SOURCE_DIR}/package.json" PACKAGE_JSON)
string(JSON PACKAGE_JSON_PROJECT_VERSION GET "${PACKAGE_JSON}" version)
string(
    REGEX REPLACE "^([0-9]+\\.[0-9]+\\.[0-9]+)-dev\\.([0-9]+)"
    "\\1.\\2"
    BASE_VERSION
    "${PACKAGE_JSON_PROJECT_VERSION}"
)
project(cocktail-maker-protocol VERSION ${BASE_VERSION} LANGUAGES CXX)
cmake_policy(SET CMP0167 NEW)
message(STATUS "Project version: ${PROJECT_VERSION}")

list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")
include(CPM)

cpmaddpackage(URI "gh:commschamp/commsdsl@8.0.1" OPTIONS "COMMSDSL_BUILD_COMMSDSL2COMMS ON" "COMMSDSL_BUILD_UNIT_TESTS OFF" "BUILD_TESTING OFF")
add_custom_command(
    OUTPUT ${CMAKE_CURRENT_SOURCE_DIR}/generated/CMakeLists.txt
    BYPRODUCTS
        ${CMAKE_CURRENT_SOURCE_DIR}/generated/include/proto/Version.h
        ${CMAKE_CURRENT_SOURCE_DIR}/generated/include/proto/MsgId.h
        ${CMAKE_CURRENT_SOURCE_DIR}/protocol/include/proto/FrameInterface.h
    COMMAND
        $<TARGET_FILE:commsdsl2comms> -o ${CMAKE_CURRENT_SOURCE_DIR}/generated
        ${CMAKE_CURRENT_SOURCE_DIR}/schema.xml
    DEPENDS commsdsl2comms ${CMAKE_CURRENT_SOURCE_DIR}/schema.xml
    VERBATIM
)

add_custom_target(
    generate_commsdsl2comms
    ALL
    DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/generated/CMakeLists.txt
)
