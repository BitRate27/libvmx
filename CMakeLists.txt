cmake_minimum_required(VERSION 3.28)

project(vmxcodec LANGUAGES CXX)

# Sources (relative to this CMakeLists.txt)
set(SOURCES
  ../src/vmxcodec_x86.cpp
  ../src/vmxcodec_avx2.cpp
  ../src/vmxcodec.cpp
)

add_library(vmx SHARED ${SOURCES})

# Match the reference output name: libvmx.dll on Windows
set_target_properties(vmx PROPERTIES OUTPUT_NAME "libvmx")

# C++ standard
set_target_properties(vmx PROPERTIES
  CXX_STANDARD 17
  CXX_STANDARD_REQUIRED ON
)

# Compiler flags similar to the reference clang++ command
target_compile_options(vmx PRIVATE
  # Clang/LLVM flags
  $<$<CXX_COMPILER_ID:Clang>:-O3>
  $<$<CXX_COMPILER_ID:Clang>:-fdeclspec>
  $<$<CXX_COMPILER_ID:Clang>:-mlzcnt>
  $<$<CXX_COMPILER_ID:Clang>:-mavx2>
  $<$<CXX_COMPILER_ID:Clang>:-mbmi>

  # GNU-ish fallback (may or may not support all flags)
  $<$<CXX_COMPILER_ID:GNU>:-O3>
  $<$<CXX_COMPILER_ID:GNU>:-fdeclspec>
  $<$<CXX_COMPILER_ID:GNU>:-mlzcnt>
  $<$<CXX_COMPILER_ID:GNU>:-mavx2>
  $<$<CXX_COMPILER_ID:GNU>:-mbmi>
)

# Linker flag to pass the exports.def on Windows.
# For clang-cl / clang++ on Windows we forward to the linker with -Wl,/DEF:...
# For MSVC link the /DEF flag directly.
if(WIN32)
  if(CMAKE_CXX_COMPILER_ID MATCHES "Clang")
    target_link_options(vmx PRIVATE "-Wl,/DEF:../exports.def")
  else()
    target_link_options(vmx PRIVATE "/DEF:../exports.def")
  endif()
endif()

# Optional: set visibility/export macros if needed
# target_compile_definitions(vmx PRIVATE VMX_BUILD_DLL)

# Note: ensure exports.def exists next to this CMakeLists.txt (or give full path)