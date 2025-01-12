#/* CMakeLists.txt */
#/* Project builder */

PROJECT(pba)

CMAKE_MINIMUM_REQUIRED(VERSION 3.0)

IF(COMMAND CMAKE_POLICY)
     CMAKE_POLICY(SET CMP0016 NEW)
ENDIF(COMMAND CMAKE_POLICY)

set(CMAKE_BUILD_TYPE "Release")
SET(CMAKE_SKIP_BUILD_RPATH TRUE)
SET(CMAKE_BUILD_WITH_INSTALL_RPATH FALSE)
SET(CMAKE_INSTALL_RPATH_USE_LINK_PATH FALSE)

SET(CMAKE_MODULE_PATH "cmake/;${CMAKE_MODULE_PATH}")

SET(EXECUTABLE_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/bin)
SET(LIBRARY_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/lib)

# Set a default build type if none was specified
IF(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
  MESSAGE(STATUS "Setting build type to 'Release' as none was specified.")
  SET(CMAKE_BUILD_TYPE Release CACHE STRING "Choose the type of build." FORCE)
  # Set the possible values of build type for cmake-gui
  SET_PROPERTY(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS "Debug" "Release"
    "MinSizeRel" "RelWithDebInfo")
ENDIF()

if(MSVC)
    SET(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${CMAKE_C_FLAGS_RELEASE_INIT}")
    SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${CMAKE_CXX_FLAGS_RELEASE_INIT}")
else()
    SET(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${CMAKE_C_FLAGS_RELEASE_INIT} -mfpmath=sse -fPIC -Wno-error=reorder -fpermissive -Wno-narrowing")
    SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${CMAKE_CXX_FLAGS_RELEASE_INIT} -mfpmath=sse -fPIC -Wno-error=reorder -fpermissive -Wno-narrowing")
endif()

####
# find threads
find_package(Threads REQUIRED)
IF(CMAKE_USE_PTHREADS_INIT)
  SET(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -pthread")
  SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -pthread")
ENDIF()

#####
# find CUDA
FIND_PACKAGE(CUDA REQUIRED)

IF(CUDA_FOUND)
  ADD_DEFINITIONS(-DPBA_DLL)
ELSE()
  ADD_DEFINITIONS(-DPBA_NO_GPU)
ENDIF()

OPTION(CPUPBA_USE_AVX "USE_AVX" ON)

if(CPUPBA_USE_AVX)
  ADD_DEFINITIONS(-DCPUPBA_USE_AVX)
  SET(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -mavx")
  SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -mavx")
endif()

ADD_SUBDIRECTORY(src/pba)
ADD_SUBDIRECTORY(src/driver)

INSTALL(
  DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/src/pba
  DESTINATION include/
  COMPONENT headers
  FILES_MATCHING PATTERN "*.hpp" PATTERN "*.h"
)

message(STATUS)
message(STATUS "General configuration for ${PROJECT_NAME}")
message(STATUS "======================================================================")
message(STATUS "Platform:")
message(STATUS "  Arch / OS:            ${CMAKE_HOST_SYSTEM_PROCESSOR} / ${CMAKE_HOST_SYSTEM_NAME} ${CMAKE_HOST_SYSTEM_VERSION}")
message(STATUS "  C/C++ Compiler:       ${CMAKE_CXX_COMPILER_ID} ${CMAKE_CXX_COMPILER_VERSION}")
message(STATUS "  Build type:           ${CMAKE_BUILD_TYPE}")
message(STATUS "======================================================================")
message(STATUS)
