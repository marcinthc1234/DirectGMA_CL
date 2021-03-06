# ########################################################################
# Copyright 2016 Advanced Micro Devices, Inc.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
# http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
# ########################################################################

# We require 2.8.12 for linking libraries we specify PRIVATE, INTERFACE and
# PUBLIC keywords
cmake_minimum_required( VERSION 2.8.12 )

project( DGMA_CL )
message( STATUS "Configuring DGMA_CL" )

if( CMAKE_GENERATOR MATCHES "NMake" )
	option( NMAKE_COMPILE_VERBOSE "Print VERBOSE compile/link msgs to the console" OFF )
	if( NMAKE_COMPILE_VERBOSE )
		set( CMAKE_START_TEMP_FILE "" )
		set( CMAKE_END_TEMP_FILE "" )
		set( CMAKE_VERBOSE_MAKEFILE 1 )
	endif( )
endif( )

if( MSVC_IDE )
  set( BUILD64 ${CMAKE_CL_64} )
  set_property( GLOBAL PROPERTY USE_FOLDERS TRUE )
else( )
  option( BUILD64 "Build a 64-bit product" ON )
endif( )

# Modify the global find property to help us find libraries like Boost in the correct paths for 64-bit
# Essentially, find_library calls will look for /lib64 instead of /lib; works for windows and linux
if( BUILD64 )
  set_property( GLOBAL PROPERTY FIND_LIBRARY_USE_LIB64_PATHS TRUE )
  message( STATUS "64bit build - FIND_LIBRARY_USE_LIB64_PATHS TRUE" )
else( )
  set_property( GLOBAL PROPERTY FIND_LIBRARY_USE_LIB64_PATHS FALSE )
  message( STATUS "32bit build - FIND_LIBRARY_USE_LIB64_PATHS FALSE" )
endif( )




# We require OpenCL
# A standard FindOpenCL.cmake module ships with cmake 3.1, but we supply our own until 3.1 becomes more prevalent
SET( CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/cmake;${PROJECT_SOURCE_DIR}/../cmake;${CMAKE_MODULE_PATH} )
MESSAGE( STATUS "CMAKE_MODULE_PATH=${CMAKE_MODULE_PATH}" )
FIND_PACKAGE( OpenCL REQUIRED )
MESSAGE( STATUS "OPENCL_INCLUDE_DIRS=${OPENCL_INCLUDE_DIRS}" )
IF(CMAKE_COMPILER_IS_GNUCXX)
  FIND_PACKAGE( Threads REQUIRED )
ENDIF(CMAKE_COMPILER_IS_GNUCXX)

SET( dgma_cl.src
  main.cpp
)



INCLUDE_DIRECTORIES(
  ${OPENCL_INCLUDE_DIRS}
)



MESSAGE( STATUS "PROJECT_SOURCE_DIR=${PROJECT_SOURCE_DIR}" )


ADD_EXECUTABLE( DGMA_CL  ${dgma_cl.src}  )
IF(CMAKE_COMPILER_IS_GNUCXX)
  TARGET_LINK_LIBRARIES( DGMA_CL ${OPENCL_LIBRARIES}  )
ELSE(CMAKE_COMPILER_IS_GNUCXX)
  TARGET_LINK_LIBRARIES( DGMA_CL ${OPENCL_LIBRARIES} )
ENDIF(CMAKE_COMPILER_IS_GNUCXX)
SET_TARGET_PROPERTIES( DGMA_CL PROPERTIES RUNTIME_OUTPUT_DIRECTORY "${PROJECT_BINARY_DIR}/staging" )



