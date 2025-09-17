cmake_minimum_required(VERSION 3.16.0 FATAL_ERROR)

# Set the project version and language
project(Spaghettify VERSION 0.1.0 LANGUAGES C CXX ASM)
include(FetchContent)

set(NATO_PHONETIC_ALPHABET
  "Alfa" "Bravo" "Charlie" "Delta" "Echo" "Foxtrot" "Golf" "Hotel"
  "India" "Juliett" "Kilo" "Lima" "Mike" "November" "Oscar" "Papa"
  "Quebec" "Romeo" "Sierra" "Tango" "Uniform" "Victor" "Whiskey"
  "Xray" "Yankee" "Zulu"
)

execute_process(
  COMMAND git config --global --add safe.directory ${CMAKE_SOURCE_DIR}
  WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
  OUTPUT_QUIET
)

execute_process(
  COMMAND git describe --tags
  WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
  OUTPUT_VARIABLE PROJECT_VERSION
  OUTPUT_STRIP_TRAILING_WHITESPACE
)
if (DEFINED PROJECT_VERSION AND NOT "${PROJECT_VERSION}" STREQUAL "")
  string(REPLACE "-" ";" PROJECT_VERSION_LIST "${PROJECT_VERSION}")
  list(GET PROJECT_VERSION_LIST 0 PROJECT_VERSION_PATCH)
  string(REPLACE "." ";" PROJECT_VERSION_LIST "${PROJECT_VERSION_PATCH}")
  list(LENGTH PROJECT_VERSION_LIST PROJECT_VERSION_LIST_LENGTH)
  if (${PROJECT_VERSION_LIST_LENGTH} LESS 3)
    set(PROJECT_VERSION_PATCH 0)
  else()
    list(GET PROJECT_VERSION_LIST 2 PROJECT_VERSION_PATCH)
  endif()
else()
  set(PROJECT_VERSION "Unknown")
  execute_process(
    COMMAND git log --pretty=format:%h -1
    WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
    OUTPUT_VARIABLE PROJECT_VERSION_PATCH
    OUTPUT_STRIP_TRAILING_WHITESPACE
  )
  if (DEFINED PROJECT_VERSION_PATCH AND NOT "${PROJECT_VERSION_PATCH}" STREQUAL "")
    set(PROJECT_VERSION "${PROJECT_VERSION}-${PROJECT_VERSION_PATCH}")
  endif()
  set(PROJECT_VERSION_PATCH 0)
endif()

# Get the patch version number from the project version
math(EXPR PATCH_INDEX "${PROJECT_VERSION_PATCH}")

# Use the patch number to select the correct word
list(GET NATO_PHONETIC_ALPHABET ${PATCH_INDEX} PROJECT_PATCH_WORD)

set(PROJECT_BUILD_NAME "Alfredo ${PROJECT_PATCH_WORD}" CACHE STRING "" FORCE)
set(PROJECT_TEAM "github.com/harbourmasters" CACHE STRING "" FORCE)

message("Spaghetti Kart version: ${PROJECT_VERSION} Alfredo ${PROJECT_PATCH_WORD}")

# Configure the C/C++ standards used by the project
set(CMAKE_CXX_STANDARD 20 CACHE STRING "The C++ standard to use")
set(CMAKE_C_STANDARD 11 CACHE STRING "The C standard to use")
set(PROJECT_TEAM "MegaMech")
#add_compile_options(-fsanitize=address)
#add_link_options(-fsanitize=address)

# Add a custom module path to locate additional CMake modules
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/cmake/modules/")
set(YAML_CPP_STATIC_DEFINE ON)
add_compile_definitions(YAML_CPP_STATIC_DEFINE)
include(cmake/lus-cvars.cmake)

execute_process(COMMAND ${CMAKE_COMMAND} -E copy_if_different "${CMAKE_SOURCE_DIR}/config.yml" "${CMAKE_BINARY_DIR}/config.yml")
execute_process(COMMAND ${CMAKE_COMMAND} -E copy_directory "${CMAKE_SOURCE_DIR}/yamls/" "${CMAKE_BINARY_DIR}/yamls/")

set(CPP "${CMAKE_C_COMPILER}" "-E" "-P" "-Wno-trigraphs" "-x" "c")

################################################################################
# Compiler specific configuration types
################################################################################
if ("${CMAKE_CXX_COMPILER_ID}" MATCHES "GNU|Clang|AppleClang")
    set(COMMON_FLAGS "-Wall -Wextra -Wno-error \
            -Wno-return-type \
            -Wno-unused-parameter \
            -Wno-unused-function \
            -Wno-unused-variable \
            -Wno-missing-field-initializers \
            -Wno-parentheses \
            -Wno-narrowing \
            -Wno-missing-braces \
            -ffast-math \
            -flto=auto \
            -pipe"
    )

    set(COMMON_C_FLAGS "${COMMON_FLAGS} -Wno-int-conversion -Werror-implicit-function-declaration -Wno-incompatible-pointer-types")
    set(COMMON_CXX_FLAGS "${COMMON_FLAGS} -Wno-c++11-narrowing -Wno-deprecated-enum-enum-conversion -fpermissive -fomit-frame-pointer")
else()
    set(COMMON_C_FLAGS "")
    set(COMMON_CXX_FLAGS "")
endif()

################################################################################
# Global configuration types
################################################################################
set(CMAKE_C_FLAGS_DEBUG "-g -DNDEBUG ${COMMON_C_FLAGS}")
set(CMAKE_CXX_FLAGS_DEBUG "-g -DNDEBUG ${COMMON_CXX_FLAGS}")
set(CMAKE_C_FLAGS_RELEASE "-O3 -DNDEBUG ${COMMON_C_FLAGS}")
set(CMAKE_CXX_FLAGS_RELEASE "-O3 -DNDEBUG ${COMMON_CXX_FLAGS}")

# Set game compilation version
set(VERSION us)
set(USE_NETWORKING ON)
set(SKIP_XCODE_VERSION_CHECK ON)

# TODO: Sorry i broke this
set(GFX_DEBUG_DISASSEMBLER OFF)

# Add compile definitions for the target
add_compile_definitions(
    VERSION_US=1
    ENABLE_RUMBLE=1
    F3DEX_GBI=1
    _LANGUAGE_C
    _USE_MATH_DEFINES
    CIMGUI_DEFINE_ENUMS_AND_STRUCTS
    NON_MATCHING=1
    NON_EQUIVALENT=1
	AVOID_UB=1
	SPAGHETTI_VERSION="${PROJECT_VERSION}"
)

# Include directories
include_directories(
    ${CMAKE_CURRENT_SOURCE_DIR}
    ${CMAKE_CURRENT_SOURCE_DIR}/include
    ${CMAKE_CURRENT_SOURCE_DIR}/include/assets
    ${CMAKE_CURRENT_SOURCE_DIR}/src
    ${CMAKE_CURRENT_SOURCE_DIR}/src/racing
    ${CMAKE_CURRENT_SOURCE_DIR}/src/ending
    ${CMAKE_CURRENT_SOURCE_DIR}/src/data
    ${CMAKE_CURRENT_SOURCE_DIR}/src/buffers
    ${CMAKE_CURRENT_SOURCE_DIR}/src/audio
    ${CMAKE_CURRENT_SOURCE_DIR}/src/debug
    ${CMAKE_CURRENT_SOURCE_DIR}/src/networking
    ${CMAKE_CURRENT_SOURCE_DIR}/src/engine
    ${CMAKE_CURRENT_SOURCE_DIR}/src/engine/courses
    ${CMAKE_CURRENT_SOURCE_DIR}/src/engine/objects
    ${CMAKE_CURRENT_SOURCE_DIR}/src/engine/particles
    ${CMAKE_CURRENT_SOURCE_DIR}/src/enhancements
    ${CMAKE_CURRENT_SOURCE_DIR}/src/enhancements/freecam
)

# Collect source files to build the executable
file(GLOB ALL_FILES_ROOT RELATIVE ${CMAKE_CURRENT_SOURCE_DIR}
    "src/*.c"
    "src/*.h"
    "src/os/guLookAtF.c"
    "src/os/guPerspectiveF.c"
    "src/os/guMtxF2L.c"
    "src/os/guRotateF.c"
    "src/os/guOrthoF.c"
    "src/os/guScaleF.c"
    "src/os/guTranslateF.c"
    "src/os/guMtxCatF.c"
    "src/os/guMtxCatL.c"
    "src/os/guNormalize.c"
)

file(GLOB_RECURSE ALL_FILES RELATIVE ${CMAKE_CURRENT_SOURCE_DIR}
    "src/buffers/*.c"
    "src/buffers/*.h"
    "src/data/*.c"
    "src/data/*.h"
    "src/debug/*.c"
    "src/debug/*.h"
    "src/ending/*.c"
    "src/ending/*.h"
    "src/racing/*.c"
    "src/racing/*.h"
    "src/audio/*.c"
    "src/audio/*.h"
    "src/port/*.h"
    "src/port/*.c"
    "src/port/*.cpp"
    "assets/code/*.c"
    "courses/*.c"
    "courses/*.h"
    "src/networking/*.c"
    "src/networking/*.h"
    "src/engine/*.cpp"
    "src/engine/*.h"
    "src/engine/courses/*.c"
    "src/engine/courses/*.h"
    "src/engine/objects/*.cpp"
    "src/engine/objects/*.h"
    "src/engine/particles/*.cpp"
    "src/engine/particles/*.h"
    "src/enhancements/*.c"
    "src/enhancements/*.h"
    "src/enhancements/freecam/*.c"
    "src/enhancements/freecam/*.cpp"
    "src/enhancements/freecam/*.h"
    "src/engine/*.cpp"
    "src/engine/*.h"
)

list(APPEND ALL_FILES ${ALL_FILES_ROOT})

# Exclude specific files from the ALL_FILES list
list(FILTER ALL_FILES EXCLUDE REGEX ".*.inc.c")
list(FILTER ALL_FILES EXCLUDE REGEX "./src/debug/crash_screen_enhancement.c")

if(ANDROID)
    add_library(${PROJECT_NAME} SHARED ${ALL_FILES})
else()
    message(FATAL_ERROR "This Android port does not support building for other platforms.")
endif()

 #=================== SSE2NEON ===================
set(SSE2NEON_DIR ${CMAKE_BINARY_DIR}/_deps/sse2neon)
file(DOWNLOAD "https://raw.githubusercontent.com/DLTcollab/sse2neon/refs/heads/master/sse2neon.h" "${SSE2NEON_DIR}/sse2neon.h")

include_directories(${SSE2NEON_DIR})

#=================== DRLibs ===================
FetchContent_Declare(
    dr_libs
    GIT_REPOSITORY https://github.com/mackron/dr_libs.git
	GIT_TAG da35f9d6c7374a95353fd1df1d394d44ab66cf01
)
FetchContent_MakeAvailable(dr_libs)

#=================== STB ===================
include_directories(${STB_DIR})

#==============================================================================#
# Libultraship Integration                                                     #
#==============================================================================#

# Removes MPQ/OTR support
set(EXCLUDE_MPQ_SUPPORT TRUE CACHE BOOL "")
add_compile_definitions(EXCLUDE_MPQ_SUPPORT)

if(ANDROID)
    find_package(SDL2)
endif()

include_directories(
  ${CMAKE_CURRENT_SOURCE_DIR}
  ${CMAKE_CURRENT_SOURCE_DIR}/src
  ${CMAKE_CURRENT_BINARY_DIR}
  ${CMAKE_CURRENT_SOURCE_DIR}/libultraship/include
  ${CMAKE_CURRENT_SOURCE_DIR}/libultraship/include/libultraship
  ${CMAKE_CURRENT_SOURCE_DIR}/libultraship/src
  ${CMAKE_CURRENT_SOURCE_DIR}/libultraship/src/log
  ${CMAKE_CURRENT_SOURCE_DIR}/libultraship/src/debug
  ${CMAKE_CURRENT_SOURCE_DIR}/libultraship/src/menu
  ${CMAKE_CURRENT_SOURCE_DIR}/libultraship/src/utils
  ${CMAKE_CURRENT_SOURCE_DIR}/libultraship/src/utils/binarytools
  ${CMAKE_CURRENT_SOURCE_DIR}/libultraship/src/config
  ${CMAKE_CURRENT_SOURCE_DIR}/libultraship/src/resource
  ${CMAKE_CURRENT_SOURCE_DIR}/libultraship/src/resource/type
  ${CMAKE_CURRENT_SOURCE_DIR}/libultraship/src/resource/factory
  ${CMAKE_CURRENT_SOURCE_DIR}/libultraship/src/audio
  ${CMAKE_CURRENT_SOURCE_DIR}/libultraship/src/window
  ${CMAKE_CURRENT_SOURCE_DIR}/libultraship/src/window/gui
  ${CMAKE_CURRENT_SOURCE_DIR}/libultraship/src/config
  ${CMAKE_CURRENT_SOURCE_DIR}/libultraship/src/public
  ${CMAKE_CURRENT_SOURCE_DIR}/libultraship/src/public/libultra
  ${CMAKE_CURRENT_SOURCE_DIR}/libultraship/src/public/bridge
  ${CMAKE_CURRENT_SOURCE_DIR}/libultraship/extern
  ${CMAKE_CURRENT_SOURCE_DIR}/libultraship/extern/tinyxml2
  ${CMAKE_CURRENT_SOURCE_DIR}/libultraship/libultraship/Lib/
  ${CMAKE_CURRENT_SOURCE_DIR}/libultraship/libultraship/Lib/libjpeg/include/
  ${CMAKE_CURRENT_SOURCE_DIR}/libultraship/libultraship/Lib/spdlog/include/
  ${CMAKE_CURRENT_SOURCE_DIR}/libultraship/src/graphic/Fast3D/U64/PR
  ${CMAKE_CURRENT_SOURCE_DIR}/libultraship/src/graphic
  ${SDL2_INCLUDE_DIRS}
  ${GLEW_INCLUDE_DIRS}
  ${dr_libs_SOURCE_DIR}
)

add_subdirectory(libultraship ${CMAKE_CURRENT_SOURCE_DIR}/libultraship)
add_dependencies(${PROJECT_NAME} libultraship)
target_link_libraries(${PROJECT_NAME} PRIVATE libultraship)

target_compile_definitions(${PROJECT_NAME} PRIVATE
    $<$<CONFIG:Debug>:_DEBUG>
    $<$<CONFIG:Release>:NDEBUG>
    $<$<BOOL:${BUILD_CROWD_CONTROL}>:ENABLE_CROWD_CONTROL>
    SPDLOG_ACTIVE_LEVEL=0
)

set(THREADS_PREFER_PTHREAD_FLAG ON)
find_package(Threads REQUIRED)
set(ADDITIONAL_LIBRARY_DEPENDENCIES
    "libultraship;"
    SDL2::SDL2
    Threads::Threads
)

if(USE_NETWORKING)
    include_directories(${SDL2_NET_INCLUDE_DIRS})
    target_compile_definitions(${PROJECT_NAME} PRIVATE USE_NETWORKING)
    target_link_libraries(${PROJECT_NAME} PRIVATE SDL2_net)
endif()

option(USE_STANDALONE "Build as a standalone executable" OFF)
option(BUILD_STORMLIB "Build with StormLib support" OFF)

option(BUILD_SM64 "Build with Super Mario 64 support" OFF)
option(BUILD_SF64 "Build with Star Fox 64 support" OFF)
option(BUILD_FZERO "Build with F-Zero X support" OFF)

add_subdirectory(torch)
target_link_libraries(${PROJECT_NAME} PRIVATE torch "${ADDITIONAL_LIBRARY_DEPENDENCIES}")

################################################################################
# Compile and link options
################################################################################
if (CMAKE_CXX_COMPILER_ID MATCHES "GNU|Clang|AppleClang")
    target_compile_options(${PROJECT_NAME} PRIVATE
        -O2

        -Wno-error
        -Wno-format-security
        -Wno-multichar
        -Wno-return-type
        -Wno-narrowing
        -Wno-switch-outside-range
        $<$<COMPILE_LANGUAGE:C>:
            -Werror-implicit-function-declaration
            -Wno-incompatible-pointer-types
            -Wno-discarded-array-qualifiers
            -Wno-discarded-qualifiers
            -Wno-int-conversion
            -Wno-builtin-declaration-mismatch
            -Wno-switch-unreachable
            -Wno-stringop-overflow
            -Wno-implicit-int
        >
    )
endif()

include(ExternalProject)
ExternalProject_Add(TorchExternal
    PREFIX TorchExternal
    SOURCE_DIR ${CMAKE_SOURCE_DIR}/torch
    CMAKE_ARGS -DCMAKE_INSTALL_PREFIX=${CMAKE_BINARY_DIR}/torch
)

ExternalProject_Get_Property(TorchExternal install_dir)
set(TORCH_EXECUTABLE ${install_dir}/src/TorchExternal-build/torch)

add_custom_target(
    ExtractAssets
    DEPENDS TorchExternal
    WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
    COMMAND ${TORCH_EXECUTABLE} header -o baserom.us.z64
    COMMAND ${TORCH_EXECUTABLE} o2r baserom.us.z64
    COMMAND ${TORCH_EXECUTABLE} pack assets spaghetti.o2r o2r
    COMMAND ${CMAKE_COMMAND} -E copy_if_different "${CMAKE_SOURCE_DIR}/mk64.o2r" "${CMAKE_BINARY_DIR}/mk64.o2r"
    COMMAND ${CMAKE_COMMAND} -E copy_if_different "${CMAKE_SOURCE_DIR}/spaghetti.o2r" "${CMAKE_BINARY_DIR}/spaghetti.o2r"
)

add_custom_target(
    GenerateO2R
    DEPENDS TorchExternal
    WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
    COMMAND ${TORCH_EXECUTABLE} pack assets spaghetti.o2r o2r
    COMMAND ${CMAKE_COMMAND} -E copy_if_different "${CMAKE_SOURCE_DIR}/spaghetti.o2r" "${CMAKE_BINARY_DIR}/spaghetti.o2r"
)

