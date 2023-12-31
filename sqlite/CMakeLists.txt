cmake_minimum_required(VERSION 3.4...3.18)
project(sqlite3)

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

set(CMAKE_CXX_STANDARD 11)

set(HEADER_ROOT "${PROJECT_SOURCE_DIR}")
set(HEADERS sqlite3.h sqlite3ext.h)
set(SOURCES sqlite3.c)
add_library(${PROJECT_NAME} ${SOURCES})
install(
  TARGETS ${PROJECT_NAME}
  LIBRARY DESTINATION lib
  ARCHIVE DESTINATION lib)
foreach(file ${HEADERS})
  get_filename_component(dir ${file} DIRECTORY)
  install(FILES ${file} DESTINATION include)
endforeach()

add_executable(sqlite3_exe shell.c)
target_link_libraries(sqlite3_exe ${PROJECT_NAME} pthread dl)
set_target_properties(sqlite3_exe PROPERTIES OUTPUT_NAME sqlite3)
install(TARGETS sqlite3_exe RUNTIME DESTINATION bin)
