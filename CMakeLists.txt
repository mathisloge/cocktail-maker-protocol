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

set(GENERATED_DIR "${CMAKE_CURRENT_SOURCE_DIR}/generated")
add_custom_command(
    OUTPUT "${GENERATED_DIR}/CMakeLists.txt"
    BYPRODUCTS
        "${GENERATED_DIR}/include/proto/Version.h"
        "${GENERATED_DIR}/include/proto/MsgId.h"
        "${GENERATED_DIR}/include/proto/FrameInterface.h"
    COMMAND "${CMAKE_COMMAND}" -E rm -rf "${GENERATED_DIR}"
    COMMAND
        $<TARGET_FILE:commsdsl2comms> -o "${GENERATED_DIR}"
        "${CMAKE_CURRENT_SOURCE_DIR}/schema.xml"
    DEPENDS commsdsl2comms "${CMAKE_CURRENT_SOURCE_DIR}/schema.xml"
    VERBATIM
)

add_custom_target(
    generate_commsdsl2comms
    ALL
    DEPENDS "${GENERATED_DIR}/CMakeLists.txt"
)
