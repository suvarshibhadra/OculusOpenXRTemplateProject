# Author: Suv Bhadra

cmake_minimum_required(VERSION 3.0)
#Rename to your app
project(MyOculusOpenXRApp)

if(POLICY CMP0075)
    cmake_policy(SET CMP0075 NEW)
endif()

set(OPENXR_ROOT_DIRECTORY "OpenXR")
add_subdirectory(${OPENXR_ROOT_DIRECTORY})

add_subdirectory(hello_xr)
