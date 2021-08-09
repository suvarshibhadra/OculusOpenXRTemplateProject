# Author: Suv Bhadra

cmake_minimum_required(VERSION 3.0)
#Rename to your app
project(MyOculusOpenXRApp)

if(POLICY CMP0075)
    cmake_policy(SET CMP0075 NEW)
endif()

# Uncomment this next commit
add_subdirectory(OpenXR)
add_subdirectory(hello_xr)
