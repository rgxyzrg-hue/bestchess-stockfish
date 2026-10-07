# Builds the Stockfish program (libstockfish.so) from the official
# Stockfish source, unmodified (../native/stockfish-<version>), with its
# default network embedded.
#
# It is an ordinary executable, run by the app as a separate process and
# driven with UCI text over stdin/stdout. It is named lib<name>.so only so
# Android installs it into the app's native library folder, the one place
# an app may run its own program from.
#
# Compiler settings follow Stockfish's own Makefile (COMP=ndk,
# ARCH=armv8 / armv7-neon / x86-64-sse41-popcnt).

cmake_minimum_required(VERSION 3.10)
project(stockfish_engine_program VERSION 1.0.0 LANGUAGES CXX)

set(STOCKFISH_VERSION "15.1" CACHE STRING "Folder suffix in ../native")
set(SF_ROOT "${CMAKE_CURRENT_SOURCE_DIR}/../native/stockfish-${STOCKFISH_VERSION}")
set(SF_SRC "${SF_ROOT}/src")
if(NOT EXISTS "${SF_SRC}/main.cpp")
  message(FATAL_ERROR "Stockfish source not found: ${SF_SRC}")
endif()

# --- Network: embedded at build time by Stockfish's incbin, which looks
# --- for the file (by its default name) in the compiler's working folder.
file(STRINGS "${SF_SRC}/evaluate.h" SF_NET_LINE
     REGEX "define EvalFileDefaultName")
string(REGEX MATCH "nn-[0-9a-f]+\\.nnue" SF_NET "${SF_NET_LINE}")
set(SF_NET_FILE "${CMAKE_CURRENT_SOURCE_DIR}/../native/nets/${SF_NET}")
if(NOT EXISTS "${SF_NET_FILE}")
  message(FATAL_ERROR "Network ${SF_NET} not found in native/nets")
endif()
configure_file("${SF_NET_FILE}" "${CMAKE_BINARY_DIR}/${SF_NET}" COPYONLY)
message(STATUS "Stockfish ${STOCKFISH_VERSION} with network ${SF_NET}")

# --- Sources
file(GLOB_RECURSE SF_SOURCES "${SF_SRC}/*.cpp")
add_executable(stockfish ${SF_SOURCES})

set_target_properties(stockfish PROPERTIES
  CXX_STANDARD 17
  CXX_STANDARD_REQUIRED ON
  OUTPUT_NAME "libstockfish"
  SUFFIX ".so")

target_include_directories(stockfish PRIVATE "${SF_SRC}")

target_compile_definitions(stockfish PRIVATE NDEBUG USE_PTHREADS)

target_compile_options(stockfish PRIVATE -O3 -fno-exceptions)

# --- Per-architecture settings (as in Stockfish's Makefile)
if(ANDROID_ABI STREQUAL "arm64-v8a")
  target_compile_definitions(stockfish PRIVATE
    IS_64BIT USE_POPCNT USE_NEON=8)
elseif(ANDROID_ABI STREQUAL "armeabi-v7a")
  target_compile_definitions(stockfish PRIVATE
    USE_POPCNT USE_NEON=7)
  target_compile_options(stockfish PRIVATE -mfpu=neon)
elseif(ANDROID_ABI STREQUAL "x86_64")
  # Emulator only. Android x86_64 guarantees SSE4.2 and POPCNT.
  target_compile_definitions(stockfish PRIVATE
    IS_64BIT USE_POPCNT USE_SSE41 USE_SSSE3 USE_SSE2)
  target_compile_options(stockfish PRIVATE -msse4.1 -mpopcnt)
endif()

# 16 KB memory pages (required by Google Play for newer Android versions).
target_link_options(stockfish PRIVATE "-Wl,-z,max-page-size=16384")
