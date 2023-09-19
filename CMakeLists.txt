cmake_minimum_required(VERSION 3.4...3.18)
project(cubao_gdal)

set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
set(CMAKE_POSITION_INDEPENDENT_CODE ON)

if(NOT CMAKE_BUILD_TYPE OR CMAKE_BUILD_TYPE STREQUAL "")
  set(CMAKE_BUILD_TYPE
      "Release"
      CACHE STRING "" FORCE)
  message(STATUS "Set build type to default: ${CMAKE_BUILD_TYPE}")
else()
  message(STATUS "Your build type: ${CMAKE_BUILD_TYPE}")
endif()
if(CMAKE_BUILD_TYPE STREQUAL "Debug")
  set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -O0 -ggdb")
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -O0 -ggdb")
elseif(CMAKE_BUILD_TYPE STREQUAL "Release")
  set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -O3")
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -O3")
endif()

set(CMAKE_CXX_STANDARD 17)
include_directories(BEFORE ${PROJECT_BINARY_DIR} ${PROJECT_SOURCE_DIR})
include_directories(${PYBIND11_INCLUDE_DIR})

set(BUILD_SHARED_LIBS OFF)

include(FetchContent)
set(ZLIB_COMPAT ON)
set(ZLIB_ENABLE_TESTS OFF)
set(ZLIBNG_ENABLE_TESTS OFF)
set(WITH_GZFILEOP ON)
# set(WITH_NEW_STRATEGIES OFF)
set(WITH_GTEST OFF)
add_subdirectory(zlib-ng)

set(PYBIND11_CPP_STANDARD -std=c++17)
add_subdirectory(pybind11)
pybind11_add_module(_pybind11_gdal pybind11.cpp)

target_compile_definitions(_pybind11_gdal
                           PRIVATE VERSION_INFO=${GDAL_VERSION_INFO})
target_link_libraries(_pybind11_gdal PRIVATE ${PROJECT_NAME})
