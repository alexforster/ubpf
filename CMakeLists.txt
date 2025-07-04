# Copyright (c) 2022-present, IO Visor Project
# SPDX-License-Identifier: Apache-2.0
#
# All rights reserved.
#
# This source code is licensed in accordance with the terms specified in
# the LICENSE file found in the root directory of this source tree.
#

include(ExternalProject)
cmake_minimum_required(VERSION 3.16)
project("ubpf")

include("cmake/platform.cmake")
include("cmake/settings.cmake")
include("cmake/options.cmake")
include("cmake/version.cmake")

if(UBPF_ENABLE_TESTS)
  include("CTest")
endif()

add_subdirectory("vm")

if(NOT UBPF_SKIP_EXTERNAL)
  ExternalProject_Add(Conformance
      INSTALL_COMMAND ""
      SOURCE_DIR ${CMAKE_SOURCE_DIR}/external/bpf_conformance
			BINARY_DIR ${CMAKE_BINARY_DIR}/external/bpf_conformance
      EXCLUDE_FROM_ALL true
      STEP_TARGETS build)
endif()

if(UBPF_ENABLE_TESTS)
  add_subdirectory("custom_tests")
  add_subdirectory("ubpf_plugin")
  add_subdirectory("bpf")
  add_subdirectory("aarch64_test")
endif()

if(UBPF_ENABLE_PACKAGE)
  include("cmake/packaging.cmake")
endif()

if (UBPF_ENABLE_LIBFUZZER)
  if (PLATFORM_WINDOWS)
    # Set compiler flags for libfuzzer
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} /fsanitize=address /fsanitize-coverage=inline-bool-flag /fsanitize-coverage=edge /fsanitize-coverage=trace-cmp /fsanitize-coverage=trace-div")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /fsanitize=address /fsanitize-coverage=inline-bool-flag /fsanitize-coverage=edge /fsanitize-coverage=trace-cmp /fsanitize-coverage=trace-div")
  endif()
  add_subdirectory("libfuzzer")
  add_subdirectory("external/ebpf-verifier")
endif()
