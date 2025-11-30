# Fuzz targets for libjpeg-turbo
# Modified for AFL++ fuzzing framework
#
# Build with: cmake -DWITH_FUZZ=ON ..
# Uses -fsanitize=fuzzer to provide main() for LLVMFuzzerTestOneInput harnesses

if(NOT ENABLE_STATIC)
  message(FATAL_ERROR "Fuzz targets require static libraries.")
endif()
if(NOT WITH_TURBOJPEG)
  message(FATAL_ERROR "Fuzz targets require the TurboJPEG API library.")
endif()

enable_language(CXX)

set(EFFECTIVE_CXX_FLAGS
  "${CMAKE_CXX_FLAGS} ${CMAKE_CXX_FLAGS_${CMAKE_BUILD_TYPE_UC}}")
message(STATUS "C++ Compiler flags = ${EFFECTIVE_CXX_FLAGS}")

# Add -fsanitize=fuzzer for libFuzzer-style harnesses
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fsanitize=fuzzer")
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fsanitize=fuzzer")

# Helper macro to add a fuzz target that links to turbojpeg-static
macro(add_turbojpeg_fuzz_target target source_file)
  add_executable(${target} ${source_file})
  target_link_libraries(${target} turbojpeg-static)
  install(TARGETS ${target} RUNTIME DESTINATION bin)
endmacro()

# Helper macro to add a fuzz target that links to jpeg-static
macro(add_jpeg_fuzz_target target source_file)
  add_executable(${target} ${source_file})
  target_link_libraries(${target} jpeg-static)
  install(TARGETS ${target} RUNTIME DESTINATION bin)
endmacro()

# cjpeg fuzzer - uses jpeg-static and extra source files
add_executable(fuzz-cjpeg cjpeg.cc ../src/cdjpeg.c
  ../src/rdbmp.c ../src/rdgif.c ../src/rdppm.c ../src/rdswitch.c
  ../src/rdtarga.c)
target_link_libraries(fuzz-cjpeg jpeg-static)
install(TARGETS fuzz-cjpeg RUNTIME DESTINATION bin)

# Decompression fuzzers (use JPEG input)
add_turbojpeg_fuzz_target(fuzz-decompress decompress.cc)
add_turbojpeg_fuzz_target(fuzz-decompress-yuv decompress_yuv.cc)

# Compression fuzzers (use raw image input)
add_turbojpeg_fuzz_target(fuzz-compress compress.cc)
add_turbojpeg_fuzz_target(fuzz-compress-yuv compress_yuv.cc)
add_turbojpeg_fuzz_target(fuzz-compress-lossless compress_lossless.cc)
add_turbojpeg_fuzz_target(fuzz-compress12 compress12.cc)
add_turbojpeg_fuzz_target(fuzz-compress12-lossless compress12_lossless.cc)
add_turbojpeg_fuzz_target(fuzz-compress16-lossless compress16_lossless.cc)

# Transform fuzzer (uses JPEG input)
add_turbojpeg_fuzz_target(fuzz-transform transform.cc)
