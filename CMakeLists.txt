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
    DOWNLOAD https://github.com/xiph/flac/archive/refs/tags/1.4.3.tar.gz
    ${PROJECT_SOURCE_DIR}/downloads/1.4.3.tar.gz
    SHOW_PROGRESS
    EXPECTED_HASH SHA256=7875d49596286055b52439ed42f044bd8ad426aa4cc5aabd96bfe7abb971d5e8
  )
  list(GET DOWNLOAD_STATUS 0 STATUS_CODE)
  if(NOT ${STATUS_CODE} EQUAL 0)
    list(GET DOWNLOAD_STATUS 1 ERROR_MESSAGE)
    file(REMOVE ${PROJECT_SOURCE_DIR}/downloads/1.4.3.tar.gz)
    message(FATAL_ERROR "Could not download Flac: ${ERROR_MESSAGE}")
  endif()

  message(STATUS "Extracting Flac sources")
  file(
    ARCHIVE_EXTRACT
    INPUT ${PROJECT_SOURCE_DIR}/downloads/1.4.3.tar.gz
    DESTINATION ${CMAKE_BINARY_DIR}/extract
  )
  file(
    RENAME
    ${CMAKE_BINARY_DIR}/extract/1.4.3
    ${PROJECT_SOURCE_DIR}/build
  )
endif()

# -------------------------------------------------------------------------------------------------

configure_file(
  "${PROJECT_SOURCE_DIR}/config.h"
  "${PROJECT_SOURCE_DIR}/build/config.h"
  COPYONLY
)

# -------------------------------------------------------------------------------------------------

set(
  sourceFiles
  "build/src/share/replaygain_analysis/replaygain_analysis.c"
  "build/src/share/grabbag/alloc.c"
  "build/src/share/grabbag/snprintf.c"
  "build/src/share/grabbag/cuesheet.c"
  "build/src/share/grabbag/picture.c"
  "build/src/share/grabbag/replaygain.c"
  "build/src/share/grabbag/seektable.c"
  "build/src/share/grabbag/file.c"
  "build/src/share/utf8/iconvert.c"
  "build/src/share/utf8/charset.c"
  #"build/src/share/utf8/makemap.c"
  "build/src/share/utf8/utf8.c"
  "build/src/share/utf8/charset_test.c"
  #"build/src/share/replaygain_synthesis/replaygain_synthesis.c"
  #"build/src/share/win_utf8_io/win_utf8_io.c"
  #"build/src/libFLAC/deduplication/lpc_compute_autocorrelation_intrin.c"
  #"build/src/libFLAC/deduplication/bitreader_read_rice_signed_block.c"
  #"build/src/libFLAC/deduplication/lpc_compute_autocorrelation_intrin_neon.c"
  #"build/src/libFLAC/deduplication/lpc_compute_autocorrelation_intrin_sse2.c"
  "build/src/libFLAC/stream_decoder.c"
  "build/src/libFLAC/crc.c"
  "build/src/libFLAC/float.c"
  "build/src/libFLAC/ogg_helper.c"
  "build/src/libFLAC/fixed.c"
  "build/src/libFLAC/format.c"
  #"build/src/libFLAC/stream_encoder_intrin_sse2.c"
  #"build/src/libFLAC/lpc_intrin_avx2.c"
  "build/src/libFLAC/bitreader.c"
  "build/src/libFLAC/stream_encoder_framing.c"
  #"build/src/libFLAC/lpc_intrin_sse41.c"
  "build/src/libFLAC/memory.c"
  #"build/src/libFLAC/fixed_intrin_sse42.c"
  "build/src/libFLAC/metadata_iterators.c"
  #"build/src/libFLAC/lpc_intrin_fma.c"
  #"build/src/libFLAC/fixed_intrin_sse2.c"
  "build/src/libFLAC/window.c"
  "build/src/libFLAC/stream_encoder.c"
  #"build/src/libFLAC/stream_encoder_intrin_avx2.c"
  #"build/src/libFLAC/lpc_intrin_sse2.c"
  "build/src/libFLAC/md5.c"
  "build/src/libFLAC/ogg_decoder_aspect.c"
  "build/src/libFLAC/lpc.c"
  "build/src/libFLAC/metadata_object.c"
  "build/src/libFLAC/fixed_intrin_ssse3.c"
  "build/src/libFLAC/bitwriter.c"
  "build/src/libFLAC/ogg_mapping.c"
  "build/src/libFLAC/bitmath.c"
  #"build/src/libFLAC/lpc_intrin_neon.c"
  #"build/src/libFLAC/fixed_intrin_avx2.c"
  #"build/src/libFLAC/stream_encoder_intrin_ssse3.c"
  "build/src/libFLAC/ogg_encoder_aspect.c"
  "build/src/libFLAC/cpu.c"
)
set(
  headerFiles
  "build/src/share/utf8/charset.h"
  "build/src/share/utf8/iconvert.h"
  "build/src/share/utf8/charmaps.h"
  "build/src/libFLAC/include/protected/all.h"
  "build/src/libFLAC/include/protected/stream_decoder.h"
  "build/src/libFLAC/include/protected/stream_encoder.h"
  "build/src/libFLAC/include/private/stream_encoder_framing.h"
  "build/src/libFLAC/include/private/macros.h"
  "build/src/libFLAC/include/private/all.h"
  "build/src/libFLAC/include/private/cpu.h"
  "build/src/libFLAC/include/private/fixed.h"
  "build/src/libFLAC/include/private/window.h"
  "build/src/libFLAC/include/private/md5.h"
  "build/src/libFLAC/include/private/lpc.h"
  "build/src/libFLAC/include/private/format.h"
  "build/src/libFLAC/include/private/float.h"
  "build/src/libFLAC/include/private/ogg_helper.h"
  "build/src/libFLAC/include/private/stream_encoder.h"
  "build/src/libFLAC/include/private/bitmath.h"
  "build/src/libFLAC/include/private/memory.h"
  "build/src/libFLAC/include/private/bitwriter.h"
  "build/src/libFLAC/include/private/bitreader.h"
  "build/src/libFLAC/include/private/crc.h"
  "build/src/libFLAC/include/private/ogg_decoder_aspect.h"
  "build/src/libFLAC/include/private/metadata.h"
  "build/src/libFLAC/include/private/ogg_encoder_aspect.h"
  "build/src/libFLAC/include/private/ogg_mapping.h"
  "build/include/FLAC/all.h"
  "build/include/FLAC/export.h"
  "build/include/FLAC/ordinals.h"
  "build/include/FLAC/stream_decoder.h"
  "build/include/FLAC/format.h"
  "build/include/FLAC/stream_encoder.h"
  "build/include/FLAC/assert.h"
  "build/include/FLAC/metadata.h"
  "build/include/FLAC/callback.h"
  "build/include/share/alloc.h"
  "build/include/share/grabbag.h"
  "build/include/share/macros.h"
  "build/include/share/grabbag/replaygain.h"
  "build/include/share/grabbag/seektable.h"
  "build/include/share/grabbag/picture.h"
  "build/include/share/grabbag/file.h"
  "build/include/share/grabbag/cuesheet.h"
  "build/include/share/endswap.h"
  "build/include/share/win_utf8_io.h"
  "build/include/share/compat.h"
  "build/include/share/getopt.h"
  "build/include/share/safe_str.h"
  "build/include/share/replaygain_synthesis.h"
  "build/include/share/utf8.h"
  "build/include/share/private.h"
  "build/include/share/replaygain_analysis.h"
)

# -------------------------------------------------------------------------------------------------

add_library(Flac STATIC)
add_library(Flac::Static ALIAS Flac)

target_compile_definitions(
  Flac
  PRIVATE HAVE_CONFIG_H
  PRIVATE FLAC_BUILD
)

target_include_directories(
  Flac
  PUBLIC "build/include"
  PUBLIC "build/src/libFLAC/include"
  PRIVATE "build"
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
