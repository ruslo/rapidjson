# Copyright (c) 2011 Milo Yip (miloyip@gmail.com)
# Copyright (c) 2013 Rafal Jeczalik (rjeczalik@gmail.com)
# Copyright (c) 2016 Ruslan Baratov
# Distributed under the MIT License (see license.txt file)

cmake_minimum_required(VERSION 3.0)
project(rapidjson VERSION 0.11)

option(RAPIDJSON_BUILD_EXAMPLES "Build rapidjson examples." OFF)
option(RAPIDJSON_BUILD_TESTS "Build rapidjson perftests and unittests." OFF)
option(RAPIDJSON_USE_SSE2 "Configure rapidjson to use SSE2." OFF)
option(
    RAPIDJSON_USE_SSE42
    "Conifgure rapidjson to use SSE4.2, overwrites SSE2."
    OFF
)

if(RAPIDJSON_USE_SSE42)
  add_definitions("-DRAPIDJSON_SSE42")
  if(MSVC)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /arch:SSE4.2")
  else()
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -msse4.2")
  endif()
else()
  if(RAPIDJSON_USE_SSE2)
    add_definitions("-DRAPIDJSON_SSE2")
    if(MSVC)
      set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /arch:SSE2")
    else()
      set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -msse2")
    endif()
  endif()
endif()

include_directories("./include")

if(RAPIDJSON_BUILD_EXAMPLES)
  add_subdirectory(example)
endif()

if(RAPIDJSON_BUILD_TESTS)
  add_subdirectory(test)
endif()

add_library(rapidjson INTERFACE)

### Install ###
set(generated_dir "${CMAKE_CURRENT_BINARY_DIR}/generated")

set(config_install_dir "lib/cmake/${PROJECT_NAME}")
set(include_install_dir "include")

set(version_config "${generated_dir}/${PROJECT_NAME}ConfigVersion.cmake")
set(project_config "${generated_dir}/${PROJECT_NAME}Config.cmake")
set(targets_export_name "${PROJECT_NAME}Targets")
set(namespace "${PROJECT_NAME}::")

include(CMakePackageConfigHelpers)
write_basic_package_version_file(
    "${version_config}" COMPATIBILITY SameMajorVersion
)

# Note: use 'targets_export_name'
configure_package_config_file(
    "cmake/Config.cmake.in"
    "${project_config}"
    INSTALL_DESTINATION "${config_install_dir}"
)

install(
    TARGETS rapidjson
    EXPORT "${targets_export_name}"
    INCLUDES DESTINATION "${include_install_dir}"
)

install(
    DIRECTORY "include/rapidjson"
    DESTINATION "${include_install_dir}"
)

install(
    FILES "${project_config}" "${version_config}"
    DESTINATION "${config_install_dir}"
)

install(
    EXPORT "${targets_export_name}"
    NAMESPACE "${namespace}"
    DESTINATION "${config_install_dir}"
)
