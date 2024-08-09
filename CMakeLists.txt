#!/usr/bin/cmake
cmake_minimum_required (VERSION 3.18)

# -------------------------------------------------------------------------------------------------

project(
  Flac
  VERSION 1.4.3
  DESCRIPTION "Popular lossless audio codec for integer-based audio data"
)

# This sets a bunch of compile flags and defined ${NUCLEX_COMPILER_TAG} to
# say something like linux-gcc-13.2-amd64-debug. You should have this directory
# if you do a full clone of a project that is using this third-party library build.
include("../../build-system/cmake/cplusplus.cmake")

# -------------------------------------------------------------------------------------------------

# TODO: Effin' CMake again. Can't this abomination of a useless build system just die?
#  If downloads the file. It complains about the checksum if it's wrong.
# Then it writes a BAD status into DOWNLOAD_STATUS. And leaves ERROR_MESSAGE empty.
# Completely useless. Weird, custom build language with side effects as the primary
# means of returning results and that probably changes between versions. Not fixing this.
if(NOT EXISTS ${PROJECT_SOURCE_DIR}/build)
  message(STATUS "Build directory not present, downloading sources")

  file(
    DOWNLOAD https://github.com/xiph/flac/archive/refs/tags/v1.4.tar.gz
    ${PROJECT_SOURCE_DIR}/downloads/v1.4.tar.gz
    SHOW_PROGRESS
    EXPECTED_HASH SHA256=7875d49596286055b52439ed42f044bd8ad426aa4cc5aabd96bfe7abb971d5e8
  )
  list(GET DOWNLOAD_STATUS 0 STATUS_CODE)
  if(NOT ${STATUS_CODE} EQUAL 0)
    list(GET DOWNLOAD_STATUS 1 ERROR_MESSAGE)
    file(REMOVE ${PROJECT_SOURCE_DIR}/downloads/v1.4.tar.gz)
    message(FATAL_ERROR "Could not download Flac: ${ERROR_MESSAGE}")
  endif()

  message(STATUS "Extracting Flac sources")
  file(
    ARCHIVE_EXTRACT
    INPUT ${PROJECT_SOURCE_DIR}/downloads/v1.4.tar.gz
    DESTINATION ${CMAKE_BINARY_DIR}/extract
  )
  file(
    RENAME
    ${CMAKE_BINARY_DIR}/extract/v1.4
    ${PROJECT_SOURCE_DIR}/build
  )
endif()

# -------------------------------------------------------------------------------------------------

#configure_file(
#	"${PROJECT_SOURCE_DIR}/config.h"
#	"${PROJECT_SOURCE_DIR}/build/config.h"
#	COPYONLY
#)

# -------------------------------------------------------------------------------------------------

set(
  sourceFiles
  "build/src/flac_encoder.c"
  "build/src/flac.c"
  "build/src/flac_multistream.c"
  "build/src/flac_projection_decoder.c"
)
set(
  headerFiles
  "build/include/flac_projection.h"
  "build/include/flac_custom.h"
  "build/include/flac_defines.h"
  "build/include/flac_multistream.h"
  "build/include/flac_types.h"
  "build/include/flac.h"
)

# -------------------------------------------------------------------------------------------------

add_library(Flac STATIC)
add_library(Flac::Static ALIAS Flac)

target_compile_definitions(
  Flac
  PRIVATE FLAC_BUILD
)

target_include_directories(
  Flac
  PUBLIC "build/include"
)

target_sources(
  Flac
  PUBLIC ${headerFiles}
  PRIVATE ${sourceFiles}
)

set_target_properties(Flac PROPERTIES PREFIX "")
set_target_properties(Flac PROPERTIES OUTPUT_NAME "flac")

# -------------------------------------------------------------------------------------------------

install(
  TARGETS Flac
  ARCHIVE DESTINATION ${PROJECT_SOURCE_DIR}/bin/${NUCLEX_COMPILER_TAG}
)

# -------------------------------------------------------------------------------------------------

file(
  WRITE "${PROJECT_SOURCE_DIR}/FlacConfig.cmake"
  "#!/usr/bin/cmake

# Configuration to include Flac in a CMake-based project. If you want to
# reference Flac as an externally compiled static library, do this:
#
#    set(Flac_DIR \"../ThirdParty/flac\")
#    find_package(Flac REQUIRED CONFIG)
#
#    target_link_libraries(
#      MyAwesomeProject
#      PRIVATE Flac::Static
#    )
#
# Alternatively, if you want to build Flac together with your project,
# use the normal CMakeLists.txt with CMake's add_subdirectory() command:
#
#    add_subdirectory(
#      \"\${PROJECT_SOURCE_DIR}/../ThirdParty/flac\"
#      \"\${CMAKE_BINARY_DIR}/flac\"
#    )
#
#    target_link_libraries(
#      MyAwesomeProject
#      PRIVATE Flac
#    )
#

# -------------------------------------------------------------------------------------------------

if(NOT DEFINED NUCLEX_COMPILER_TAG)
  message(
    FATAL_ERROR
    \"NUCLEX_COMPILER_TAG not defined! Include cplusplus.cmake before importing this package \\
    in order to generate a tag identifying the platform/compiler/architecture/variant!\"
  )
endif()

# -------------------------------------------------------------------------------------------------

if(NOT EXISTS \"\${CMAKE_CURRENT_LIST_DIR}/bin/\${NUCLEX_COMPILER_TAG}\")
  # TODO: Warn and link release build when compiling in debug mode
  # TODO: Warn and link build for older compiler version if found
  message(
    FATAL_ERROR
    \"Directory '\${CMAKE_CURRENT_LIST_DIR}/bin/\${NUCLEX_COMPILER_TAG}' not found. \\
    Please either build and install this project before importing it via \\
    find_package() or use this project's main CMakeFiles.txt via add_subdirectory()!\"
  )
endif()

# -------------------------------------------------------------------------------------------------

add_library(Flac::Static STATIC IMPORTED)

# This may cause warnings on recent GCC versions (10.0.0+?) with LTO because GCC detects
# that the headers used during build (residing in build/) are not the same used when
# linking the library (copies resising in Include/).
#
# CMake doesn't run the install step during build, so the only way to get the headers
# in place before building would be by copying them rather than installing them.
set_target_properties(
  Flac::Static PROPERTIES
  INTERFACE_INCLUDE_DIRECTORIES \"\${CMAKE_CURRENT_LIST_DIR}/Include\"
  IMPORTED_LINK_INTERFACE_LANGUAGES \"C\"
)

if(WIN32)
  set_target_properties(
    Flac::Static PROPERTIES
    IMPORTED_LOCATION \"\${CMAKE_CURRENT_LIST_DIR}/bin/\${NUCLEX_COMPILER_TAG}/flac.lib\"
  )
else()
  set_target_properties(
    Flac::Static PROPERTIES
    IMPORTED_LOCATION \"\${CMAKE_CURRENT_LIST_DIR}/bin/\${NUCLEX_COMPILER_TAG}/flac.a\"
  )
endif()

message(STATUS \"Imported Flac targets with binaries in '\${CMAKE_CURRENT_LIST_DIR}'\")")

# -------------------------------------------------------------------------------------------------
