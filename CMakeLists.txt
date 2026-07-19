cmake_minimum_required(VERSION 3.22)

option(QUEST "Build for quest" ON)

# Globals
set(CMAKE_CXX_STANDARD 23)
set(CMAKE_CXX_STANDARD_REQUIRED 23)
set(CMAKE_EXPORT_COMPILE_COMMANDS TRUE)

set(SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/src)
set(INCLUDE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/include)

add_compile_options(-frtti -fexceptions -fvisibility=hidden -fPIE -fPIC -Wno-invalid-offsetof -Werror=nonportable-include-path)

# Include. Include order matters!
include(${CMAKE_CURRENT_SOURCE_DIR}/cmake/utils.cmake)
include(${CMAKE_CURRENT_SOURCE_DIR}/cmake/qpm.cmake)

set(GIT_HEADER_FILE ${CMAKE_CURRENT_SOURCE_DIR}/include/git-info.h)
include(${CMAKE_CURRENT_SOURCE_DIR}/cmake/git.cmake)
include(${CMAKE_CURRENT_SOURCE_DIR}/cmake/write-version.cmake)

if(${CMAKE_BUILD_TYPE} STREQUAL "RELEASE" OR ${CMAKE_BUILD_TYPE} STREQUAL "RelWithDebInfo" OR ${CMAKE_BUILD_TYPE} STREQUAL "MinSizeRel")
        # Better optimizations
        add_compile_options(-O3)

        # LTO
        set(CMAKE_INTERPROCEDURAL_OPTIMIZATION TRUE)
        add_compile_options(-flto)
endif()

if(${CMAKE_BUILD_TYPE} STREQUAL "DEBUG" OR ${CMAKE_BUILD_TYPE} STREQUAL "RelWithDebInfo")
        add_compile_options(-g)
endif()

# Targets
if(QUEST)
        include(${CMAKE_CURRENT_SOURCE_DIR}/cmake/targets/quest.cmake)
endif()

# Post build
include(${CMAKE_CURRENT_SOURCE_DIR}/cmake/strip.cmake)

# stop symbols leaking
# TODO: Fix
# add_link_options(-Wl, --exclude-libs, ALL)
project( NoPromo
        VERSION ${PACKAGE_VERSION})

add_compile_definitions(MOD_ID="${CMAKE_PROJECT_NAME}")
add_compile_definitions(VERSION="${CMAKE_PROJECT_VERSION}")

# Set COMPILE_ID for qpm purposes
set(COMPILE_ID ${CMAKE_PROJECT_NAME})

# recursively get all src files
RECURSE_FILES(cpp_file_list ${SOURCE_DIR}/*.cpp)
RECURSE_FILES(c_file_list ${SOURCE_DIR}/*.c)

add_library(
        ${CMAKE_PROJECT_NAME}
        SHARED
        ${cpp_file_list}
        ${c_file_list}
)

target_include_directories(${CMAKE_PROJECT_NAME} PRIVATE ${INCLUDE_DIR})
target_include_directories(${CMAKE_PROJECT_NAME} PRIVATE ${SHARED_DIR})

# Include "mmkay" if it exists, mmkay?
if(EXISTS "${EXTERN_DIR}/includes/mmkay/shared/mmkay.cmake")
        include("${EXTERN_DIR}/includes/mmkay/shared/mmkay.cmake")
endif()

# Include user includes
include("${CMAKE_CURRENT_SOURCE_DIR}/cmake/user-includes.cmake")
