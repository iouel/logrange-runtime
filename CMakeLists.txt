cmake_minimum_required(VERSION 3.20)

# Version identity lives in the header (LOGRANGE_VERSION_* in
# include/logrange/log_math.h) so a vendored copy carries its own version.
file(READ "${CMAKE_CURRENT_SOURCE_DIR}/include/logrange/log_math.h" _lr_header)
foreach(_lr_part MAJOR MINOR PATCH)
  if(NOT _lr_header MATCHES "#define LOGRANGE_VERSION_${_lr_part} +([0-9]+)")
    message(FATAL_ERROR
      "LOGRANGE_VERSION_${_lr_part} not found in include/logrange/log_math.h")
  endif()
  set(_lr_${_lr_part} "${CMAKE_MATCH_1}")
endforeach()
unset(_lr_header)

project(LogRange VERSION ${_lr_MAJOR}.${_lr_MINOR}.${_lr_PATCH} LANGUAGES CXX)
message(STATUS "LogRange version ${PROJECT_VERSION} (from log_math.h)")

include(GNUInstallDirs)
include(CMakePackageConfigHelpers)

# PROJECT_IS_TOP_LEVEL is 3.21; this project targets 3.20.
if(CMAKE_SOURCE_DIR STREQUAL CMAKE_CURRENT_SOURCE_DIR)
  set(_lr_top_level ON)
else()
  set(_lr_top_level OFF)
endif()

option(LOGRANGE_BUILD_TESTING "Build LogRange's tests" ${_lr_top_level})

# ---------------------------------------------------------------------------
# The library. Header-only, so everything is INTERFACE.
# ---------------------------------------------------------------------------
add_library(logrange INTERFACE)
add_library(LogRange::logrange ALIAS logrange)

target_include_directories(logrange INTERFACE
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
  $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>)

# Consumers get C++17 from the target rather than having to know to set it.
target_compile_features(logrange INTERFACE cxx_std_17)

# No compile flags are imposed on consumers, deliberately. What genuinely
# breaks the accumulator is reassociating math, and a compile flag cannot
# express "not fast-math". log_math.h refuses it with #error on
# __FAST_MATH__ / _M_FP_FAST instead, which is enforceable rather than
# advisory. FMA contraction is a different flag and is harmless: measured
# bit-identical with it on or off.

# ---------------------------------------------------------------------------
# Install / package. Header-only, so this is headers + a config package.
# ---------------------------------------------------------------------------
install(TARGETS logrange EXPORT LogRangeTargets)
install(DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/include/"
        DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}")
install(EXPORT LogRangeTargets
        FILE LogRangeTargets.cmake
        NAMESPACE LogRange::
        DESTINATION "${CMAKE_INSTALL_LIBDIR}/cmake/LogRange")

configure_package_config_file(
  "${CMAKE_CURRENT_SOURCE_DIR}/cmake/LogRangeConfig.cmake.in"
  "${CMAKE_CURRENT_BINARY_DIR}/LogRangeConfig.cmake"
  INSTALL_DESTINATION "${CMAKE_INSTALL_LIBDIR}/cmake/LogRange")

# SameMajorVersion as of 1.0: the error contract is stable across 1.x, so a
# consumer asking for 1.0 must accept 1.1. If a bound ever has to move again
# it takes the major with it.
write_basic_package_version_file(
  "${CMAKE_CURRENT_BINARY_DIR}/LogRangeConfigVersion.cmake"
  VERSION ${PROJECT_VERSION}
  COMPATIBILITY SameMajorVersion)

install(FILES
  "${CMAKE_CURRENT_BINARY_DIR}/LogRangeConfig.cmake"
  "${CMAKE_CURRENT_BINARY_DIR}/LogRangeConfigVersion.cmake"
  DESTINATION "${CMAKE_INSTALL_LIBDIR}/cmake/LogRange")

if(NOT LOGRANGE_BUILD_TESTING)
  return()
endif()

# ---------------------------------------------------------------------------
# Our own build flags. Deliberately NOT on the `logrange` target: a consumer
# who does add_subdirectory() here must not inherit -Werror.
#
# -ffp-contract=off / /fp:precise are pinned here for reproducibility of the
# accuracy suite, which generates its cancellation corpus with multiply-add
# expressions that contract differently between builds and would otherwise
# yield a different dataset. It is not what makes the accumulator correct.
# ---------------------------------------------------------------------------
add_library(logrange_build_flags INTERFACE)
if(MSVC)
  target_compile_options(logrange_build_flags INTERFACE
    /W4 /WX /permissive- /fp:precise)
else()
  target_compile_options(logrange_build_flags INTERFACE
    -Wall -Wextra -Werror -ffp-contract=off)
endif()

enable_testing()

# Essential runtime tests only: public API / basic arithmetic, the two
# accumulators, and the fixed accuracy/contract scenarios. The research
# repo's adversarial bound search, chain-propagation study, and rescue-shim
# instrumentation controls are not part of this product's supported surface.
foreach(_lr_test log_math rp_accum pos_accum accuracy)
  add_executable(test_${_lr_test} tests/test_${_lr_test}.cpp)
  target_link_libraries(test_${_lr_test} PRIVATE logrange logrange_build_flags)
  add_test(NAME ${_lr_test} COMMAND test_${_lr_test})
endforeach()
