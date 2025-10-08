cmake_minimum_required(VERSION 3.28)

project(vmxcodec LANGUAGES CXX)

# Sources (relative to this CMakeLists.txt)

set(SOURCES
  ../src/vmxcodec_x86.cpp
  ../src/vmxcodec_avx2.cpp
  ../src/vmxcodec.cpp
)

# Build a static library (.lib on Windows) so it can be linked into another DLL
add_library(vmx STATIC ${SOURCES})

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

# Note: a static library does not need an exports.def file or linker export flags.
# If your code currently uses a VMX_BUILD_DLL macro to control __declspec(dllexport),
# you may want to add a different macro for static builds or update the headers.

# Optional: set visibility/export macros if needed
# When building a static library we define VMX_STATIC so headers can adjust
# import/export macros (e.g., avoid __declspec(dllimport)). Make it PUBLIC so
# consumers linking against this target also get the definition.
target_compile_definitions(vmx PUBLIC VMX_STATIC)

# Note: ensure exports.def exists next to this CMakeLists.txt (or give full path)