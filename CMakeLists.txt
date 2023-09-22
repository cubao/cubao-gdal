cmake_minimum_required(VERSION 3.4...3.18)
project(cubao_gdal)

set(CMAKE_CXX_STANDARD 14)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
set(CMAKE_POSITION_INDEPENDENT_CODE ON)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_LIBRARY_OUTPUT_DIRECTORY})
set(BUILD_SHARED_LIBS OFF)
set(BUILD_TESTING OFF)
include_directories(BEFORE ${PROJECT_BINARY_DIR} ${PROJECT_SOURCE_DIR})

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

if(NOT CMAKE_INSTALL_PREFIX)
  set(CMAKE_INSTALL_PREFIX ${CMAKE_BINARY_DIR}/install)
endif()

set(ENV{PATH} "${CMAKE_INSTALL_PREFIX}/bin:$ENV{PATH}")

macro(print_include_directories)
  get_property(
    dirs
    DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
    PROPERTY INCLUDE_DIRECTORIES)
  message(STATUS "all include directories:")
  foreach(dir ${dirs})
    message(STATUS "-   ${dir}")
  endforeach()
endmacro()

macro(print_all_linked_libraries target)
  get_target_property(libs ${target} LINK_LIBRARIES)
  message(STATUS "all linked libraries: (against ${target})")
  foreach(lib ${libs})
    message(STATUS "-   ${lib}")
  endforeach()
endmacro()

macro(print_all_variables)
  get_cmake_property(vars VARIABLES)
  list(SORT vars)
  message(STATUS "all variables:")
  foreach(var ${vars})
    message(STATUS "-   ${var}=${${var}}")
  endforeach()
endmacro()

set(EXTERNAL_PROJECT_ADD_CMAKE_ARGS
    -DCMAKE_INSTALL_PREFIX=${CMAKE_INSTALL_PREFIX}
    -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE}
    -DBUILD_SHARED_LIBS=${BUILD_SHARED_LIBS}
    -DCMAKE_EXPORT_COMPILE_COMMANDS=${CMAKE_EXPORT_COMPILE_COMMANDS}
    -DCMAKE_POSITION_INDEPENDENT_CODE=${CMAKE_POSITION_INDEPENDENT_CODE}
    -DBUILD_TESTING=${BUILD_TESTING}
    #
    -DCMAKE_EXPORT_COMPILE_COMMANDS=ON
    -DCMAKE_POSITION_INDEPENDENT_CODE=ON
    #
    -DCMAKE_C_COMPILER=${CMAKE_C_COMPILER}
    -DCMAKE_C_COMPILER_TARGET=${CMAKE_C_COMPILER_TARGET}
    -DCMAKE_C_FLAGS=${CMAKE_C_FLAGS}
    -DCMAKE_CXX_COMPILER=${CMAKE_CXX_COMPILER}
    -DCMAKE_CXX_COMPILER_TARGET=${CMAKE_CXX_COMPILER_TARGET}
    -DCMAKE_CXX_FLAGS=${CMAKE_CXX_FLAGS}
    -DCMAKE_LINKER=${CMAKE_LINKER}
    -DCMAKE_AR=${CMAKE_AR}
    -DCMAKE_SYSTEM_NAME=${CMAKE_SYSTEM_NAME}
    -DCMAKE_SYSTEM_PROCESSOR=${CMAKE_SYSTEM_PROCESSOR}
    -DCMAKE_TOOLCHAIN_FILE=${TOOLCHAIN_REAL_PATH})

include(ExternalProject)
ExternalProject_Add(
  zlib
  PREFIX ${CMAKE_CURRENT_BINARY_DIR}
  SOURCE_DIR ${PROJECT_SOURCE_DIR}/zlib-ng
  CMAKE_ARGS ${EXTERNAL_PROJECT_ADD_CMAKE_ARGS} #
             -DZLIB_COMPAT=ON #
             -DZLIB_ENABLE_TESTS=OFF #
             -DZLIBNG_ENABLE_TESTS=OFF #
             -DWITH_GZFILEOP=ON #
             -DWITH_GTEST=OFF)

ExternalProject_Add(
  sqlite3
  PREFIX ${CMAKE_CURRENT_BINARY_DIR}
  SOURCE_DIR ${PROJECT_SOURCE_DIR}/sqlite
  CMAKE_ARGS ${EXTERNAL_PROJECT_ADD_CMAKE_ARGS})

ExternalProject_Add(
  proj
  DEPENDS sqlite3
  PREFIX ${CMAKE_CURRENT_BINARY_DIR}
  SOURCE_DIR ${PROJECT_SOURCE_DIR}/PROJ
  CMAKE_ARGS ${EXTERNAL_PROJECT_ADD_CMAKE_ARGS} -DNLOHMANN_JSON_ORIGIN=internal
             -DENABLE_TIFF=OFF -DENABLE_CURL=OFF -DBUILD_EXAMPLES=OFF
             -DBUILD_PROJSYNC=OFF)

ExternalProject_Add(
  gdal
  DEPENDS proj
  PREFIX ${CMAKE_CURRENT_BINARY_DIR}
  SOURCE_DIR ${PROJECT_SOURCE_DIR}/gdal
  CMAKE_ARGS ${EXTERNAL_PROJECT_ADD_CMAKE_ARGS}
             # gdal/gdal.cmake
             -DBUILD_PYTHON_BINDINGS=OFF
             -DGDAL_HIDE_INTERNAL_SYMBOLS=ON
             -DGDAL_OBJECT_LIBRARIES_POSITION_INDEPENDENT_CODE=ON
             -DGDAL_ENABLE_PLUGINS_NO_DEPS=ON
             #
             -DGDAL_SET_INSTALL_RELATIVE_RPATH=ON
             #
             -DGDAL_USE_GEOTIFF_INTERNAL=ON
             -DGDAL_USE_GIF_INTERNAL=ON
             -DGDAL_USE_JPEG_INTERNAL=ON
             -DGDAL_USE_JSONC_INTERNAL=ON
             -DGDAL_USE_LERC_INTERNAL=ON
             -DGDAL_USE_PNG_INTERNAL=ON
             -DGDAL_USE_SHAPELIB_INTERNAL=ON
             -DGDAL_USE_TIFF_INTERNAL=ON
             -DGDAL_USE_ZLIB_INTERNAL=ON
             #
             -DBUILD_DOCS=OFF)

set(PYBIND11_CPP_STANDARD -std=c++14)
add_subdirectory(pybind11)
include_directories(${PYBIND11_INCLUDE_DIR})
pybind11_add_module(_pybind11_gdal pybind11.cpp)
target_compile_definitions(_pybind11_gdal
                           PRIVATE VERSION_INFO=${GDAL_VERSION_INFO})
