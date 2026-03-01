# =============================================================================
# cmake-format: off
# SPDX-FileCopyrightText: Copyright (c) 2022-2025, NVIDIA CORPORATION.
# SPDX-License-Identifier: Apache-2.0
# cmake-format: on
# =============================================================================

cmake_minimum_required(VERSION 3.30.4 FATAL_ERROR)

include(../../cmake/rapids_config.cmake)

project(
  rmm-python
  VERSION "${RAPIDS_VERSION}"
  LANGUAGES CXX)

# On Windows, CMake's FindCUDAToolkit searches only lib/x64 but some
# installations (e.g. conda) place libraries directly in lib/.
# Resolve the correct path before find_package(rmm) triggers the search.
if(WIN32 AND NOT CUDAToolkit_LIBRARY_DIR)
  foreach(_cuda_root "$ENV{CUDA_PATH}" "$ENV{CUDAToolkit_ROOT}")
    if(_cuda_root)
      foreach(_suffix lib/x64 lib)
        if(EXISTS "${_cuda_root}/${_suffix}/cudart.lib")
          set(CUDAToolkit_LIBRARY_DIR "${_cuda_root}/${_suffix}" CACHE PATH "")
          break()
        endif()
      endforeach()
      if(CUDAToolkit_LIBRARY_DIR)
        break()
      endif()
    endif()
  endforeach()
endif()

find_package(rmm "${RAPIDS_VERSION}" REQUIRED)

# MSVC defaults to C++14; CCCL headers (pulled in via rmm::rmm) require C++17.
set(CMAKE_CXX_STANDARD 17)

if(MSVC)
  add_compile_definitions(NOMINMAX)
endif()

include(rapids-cython-core)
rapids_cython_init()

# pass through logging level to spdlog
add_compile_definitions("RMM_LOG_ACTIVE_LEVEL=RAPIDS_LOGGER_LOG_LEVEL_${RMM_LOGGING_LEVEL}")

add_subdirectory(rmm/_cuda)
add_subdirectory(rmm/librmm)
add_subdirectory(rmm/pylibrmm)
