# Required for statically linking the MSVC runtime
set(CMAKE_USER_MAKE_RULES_OVERRIDE
   ${CMAKE_CURRENT_SOURCE_DIR}/cmake/c_flag_overrides.cmake)
set(CMAKE_USER_MAKE_RULES_OVERRIDE_CXX
   ${CMAKE_CURRENT_SOURCE_DIR}/cmake/cxx_flag_overrides.cmake)

project(OculusMinimalExampleSDK_008)
cmake_minimum_required(VERSION 2.8)
include(GenerateExportHeader)
include(cmake/defaults.cmake)

set(CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/cmake)

string( TOLOWER "${CMAKE_BUILD_TYPE}" BUILD_TYPE_LOWER )
if(BUILD_TYPE_LOWER STREQUAL "debug")
   set(RIFT_DEBUG 1)
endif()

###############################################################################
#
# Oculus VR SDK dependency 
#
set(OVR_DIR ${CMAKE_CURRENT_SOURCE_DIR}/libraries/OculusSDK/LibOVR)
# Check for the existence of the LibOVR project.  If it's missing, the 
# submodule probably hasn't been checked out.
if (NOT(EXISTS ${OVR_DIR} AND IS_DIRECTORY ${OVR_DIR}))
    message(FATAL_ERROR "Download OculusSDK Version 0.0.8") 
endif()

###############################################################################
#
# Oculus VR SDK dependency 
#
set(OVR_DIR_KERNEL ${CMAKE_CURRENT_SOURCE_DIR}/libraries/OculusSDK/LibOVRKernel)
# Check for the existence of the LibOVR project.  If it's missing, the 
# submodule probably hasn't been checked out.
if (NOT(EXISTS ${OVR_DIR_KERNEL} AND IS_DIRECTORY ${OVR_DIR_KERNEL}))
    message(FATAL_ERROR "Download OculusSDK Version 0.0.8") 
endif()

# The primary SDK artifact, a static library for Oculus access 
set(BUILD_SHARED_LIBS ON)
add_subdirectory(libraries/OculusSDK/LibOVR)
set(BUILD_SHARED_LIBS OFF)
add_subdirectory(libraries/OculusSDK/LibOVRKernel)


###############################################################################
#
# Oculus C API 
#
# This provides all the access we need to the Oculus SDK.  For some strange 
# reason the header files are in LibOVR/Src instead of LibOVR/Include.  
#
include_directories(${CMAKE_SOURCE_DIR}/libraries/OculusSDK/LibOVR/Src)
include_directories(${CMAKE_SOURCE_DIR}/libraries/OculusSDK/LibOVR/Includes)
include_directories(${CMAKE_SOURCE_DIR}/libraries/OculusSDK/LibOVRKernel/Src)
include_directories(${CMAKE_SOURCE_DIR}/libraries/OculusSDK/LibOVRKernel/Kernel)
include_directories(${CMAKE_SOURCE_DIR}/libraries/OculusSDK/LibOVRKernel/GL)
include_directories(${CMAKE_SOURCE_DIR}/libraries/OculusSDK/LibOVRKernel/Tracing)
include_directories(${CMAKE_SOURCE_DIR}/libraries/OculusSDK/LibOVRKernel/Util)

