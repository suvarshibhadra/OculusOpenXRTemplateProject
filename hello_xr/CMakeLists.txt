# Copyright (c) 2017 The Khronos Group Inc.
#
# SPDX-License-Identifier: Apache-2.0
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#
# Author:
#

set(HELPER_FOLDER "OpenXR/Helpers")

# Several files use these compile time platform switches
if(WIN32)
    add_definitions(-DXR_USE_PLATFORM_WIN32)
elseif(ANDROID)
    add_definitions(-DXR_USE_PLATFORM_ANDROID)
elseif(PRESENTATION_BACKEND MATCHES "xlib")
    add_definitions(-DXR_USE_PLATFORM_XLIB)
elseif(PRESENTATION_BACKEND MATCHES "xcb")
    add_definitions(-DXR_USE_PLATFORM_XCB)

    # TODO remove once conformance supports XCB
    set(BUILD_CONFORMANCE_TESTS OFF)
elseif(PRESENTATION_BACKEND MATCHES "wayland")
    add_definitions(-DXR_USE_PLATFORM_WAYLAND)

    # TODO remove once conformance supports Wayland
    set(BUILD_CONFORMANCE_TESTS OFF)
endif()

set(VULKAN_INCOMPATIBLE FALSE)
if(NOT VULKAN_INCOMPATIBLE)
    if(NOT CMAKE_VERSION VERSION_LESS 3.7.0)
        # Find the Vulkan headers
        find_package(Vulkan)
    endif()
    if(Vulkan_FOUND)
        add_definitions(-DXR_USE_GRAPHICS_API_VULKAN)
        message(STATUS "Enabling Vulkan support")
    elseif(BUILD_ALL_EXTENSIONS)
        message(FATAL_ERROR "Vulkan headers not found")
    endif()
endif()

if(ANDROID)
 option(BUILD_LOADER "Build loader" OFF)
 add_library(openxr_loader SHARED IMPORTED)
 set_property(
      TARGET
      openxr_loader
      PROPERTY
      IMPORTED_LOCATION
      C:/Users/suv/Documents/openxr_oculus_mobile_sdk_1014/OpenXR/Libs/Android/${ANDROID_ABI}/${CMAKE_BUILD_TYPE}/libopenxr_loader.so
  )
else()
  option(BUILD_LOADER "Build loader" ON)
endif()


####

file(GLOB LOCAL_HEADERS "*.h")
file(GLOB LOCAL_SOURCE "*.cpp")
file(GLOB VULKAN_SHADERS "vulkan_shaders/*.glsl")

# For including compiled shaders
include_directories(${CMAKE_CURRENT_BINARY_DIR})

if(ANDROID)
    add_library(hello_xr MODULE
        ${LOCAL_SOURCE}
        ${LOCAL_HEADERS}
        ${VULKAN_SHADERS}
        $<TARGET_OBJECTS:android_native_app_glue>)
    target_link_libraries(hello_xr ${ANDROID_LIBRARY} ${ANDROID_LOG_LIBRARY})
else()
    add_executable(hello_xr
        ${LOCAL_SOURCE}
        ${LOCAL_HEADERS}
        ${VULKAN_SHADERS})
endif()

source_group("Headers" FILES ${LOCAL_HEADERS})
source_group("Shaders" FILES ${VULKAN_SHADERS})

compile_glsl(run_hello_xr_glsl_compiles ${VULKAN_SHADERS})

add_dependencies(hello_xr
    generate_openxr_header
    run_hello_xr_glsl_compiles
)

target_include_directories(hello_xr
    PRIVATE
    ${PROJECT_SOURCE_DIR}/${OPENXR_ROOT_DIRECTORY}/src
    ${PROJECT_SOURCE_DIR}/${OPENXR_ROOT_DIRECTORY}/src/common

    # for OpenXR headers
    ${PROJECT_SOURCE_DIR}/${OPENXR_ROOT_DIRECTORY}/include
    ${PROJECT_BINARY_DIR}/${OPENXR_ROOT_DIRECTORY}/include

    # for helper headers
    ${PROJECT_SOURCE_DIR}/${OPENXR_ROOT_DIRECTORY}/external/include
)

if(GLSLANG_VALIDATOR AND NOT GLSLC_COMMAND)
    target_compile_definitions(hello_xr PRIVATE USE_GLSLANGVALIDATOR)
endif()

if(Vulkan_FOUND)
    target_include_directories(hello_xr
        PRIVATE
        ${Vulkan_INCLUDE_DIRS}
    )
endif()

target_link_libraries(hello_xr openxr_loader)

if(TARGET openxr-gfxwrapper)
    target_link_libraries(hello_xr openxr-gfxwrapper)
endif()
if(WIN32)
    if(MSVC)
        target_compile_definitions(hello_xr PRIVATE _CRT_SECURE_NO_WARNINGS)
        target_compile_options(hello_xr PRIVATE /Zc:wchar_t /Zc:forScope /W4 /WX)
    endif()
    target_link_libraries(hello_xr ole32)
    if(MSVC)
        # Right now can't build this on MinGW because of directxcolors, etc.
        target_link_libraries(hello_xr d3d11 d3d12 d3dcompiler dxgi)
    else()
        target_compile_definitions(hello_xr PRIVATE MISSING_DIRECTX_COLORS)
    endif()
endif()

if(Vulkan_LIBRARY)
    target_link_libraries(hello_xr ${Vulkan_LIBRARY})
endif()

if(NOT ANDROID)
    install(TARGETS hello_xr
        RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR})
    if(NOT WIN32)
        install(FILES hello_xr.1 DESTINATION ${CMAKE_INSTALL_MANDIR}/man1/ COMPONENT ManPages)
    endif()
endif()

