# Specify the minimum version for CMake

cmake_minimum_required(VERSION 2.8)

# Project's name

project(Handshaking)


set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_FLAGS -std=c++11)

if(NOT DEFINED VREP_PATH)
  MESSAGE(FATAL_ERROR "Please define VREP_PATH to build this software against your VREP version")
endif()

IF(APPLE)
include_directories(${VREP_PATH}/programming/remoteApi)
include_directories(${VREP_PATH}/programming/include)
set(C_SOURCES ${VREP_PATH}/programming/remoteApi/extApi.c ${VREP_PATH}/programming/remoteApi/extApiPlatform.c)
add_definitions(-DNON_MATLAB_PARSING=1)
add_definitions(-DMAX_EXT_API_CONNECTIONS=255)
ELSE()
#include_directories(${VREP_PATH}/programming/remoteApi)
#include_directories(${VREP_PATH}/programming/include)
#set(C_SOURCES ${VREP_PATH}/programming/remoteApi/extApi.c ${VREP_PATH}/programming/remoteApi/extApiPlatform.c)
ENDIF(APPLE)
INCLUDE(FindOpenGL)
find_package(Threads REQUIRED)

set(COMMON_SOURCES common.cpp lib-cpg/Integrator.cpp Robot/Robot.cpp Robot/Robot_VREP.cpp PIDController.cpp CPG.cpp Simulation/Simulation2.cpp)

set(GRAPHICAL_SOURCES Graphical/main.cpp)

add_executable(Graphical ${C_SOURCES} ${COMMON_SOURCES} Simulation/Simulation.cpp ${GRAPHICAL_SOURCES})
add_definitions(-pthread)
target_link_libraries(Graphical ${CMAKE_SOURCE_DIR}/../Grapher/build/libGrapher.a)


IF(APPLE)
   INCLUDE_DIRECTORIES(/System/Library/Frameworks)
   INCLUDE_DIRECTORIES(/usr/local/lib)
   find_package(glfw3 3.2 REQUIRED)
   FIND_LIBRARY(COCOA_LIBRARY Cocoa)
   FIND_LIBRARY(IOKIT_LIBRARY IOKit)
   FIND_LIBRARY(COREVIDEO_LIBRARY CoreVideo)
   FIND_LIBRARY(OpenGL_LIBRARY OpenGL)
   MARK_AS_ADVANCED (COCOA_LIBRARY
		     COREVIDEO_LIBRARY
		     IOKIT_LIBRARY
                     OpenGL_LIBRARY)
   SET(EXTRA_LIBS ${COCOA_LIBRARY} ${COREVIDEO_LIBRARY} ${IOKIT_LIBRARY} ${OpenGL_LIBRARY})
   target_link_libraries(Graphical ${EXTRA_LIBS} glfw3)
ELSE()
   find_package(OpenGL REQUIRED)
   find_package(GLEW REQUIRED)
   find_package(PkgConfig REQUIRED)
   pkg_search_module(GLFW REQUIRED glfw3)
   include_directories(${GLFW_INCLUDE_DIRS} ${OPENGL_INCLUDE_DIRS} ${GLEW_INCLUDE_DIRS})
   target_link_libraries(Graphical ${GLFW_LIBRARIES} ${OPENGL_LIBRARIES} ${GLEW_LIBRARIES} -ldl -lm ${CMAKE_THREAD_LIBS_INIT})
ENDIF (APPLE)

