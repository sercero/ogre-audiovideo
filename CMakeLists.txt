#-------------------------------------------------------------------
# This file is part of the CMake build system for OGRE Audio-Video
#  (Object-oriented Graphics Rendering Engine Audio-Video Plugin)
# For the latest info, see http://www.ogre3d.org/
#
# The contents of this file are placed in the public domain. Feel
# free to make use of it in any way you like.
#-------------------------------------------------------------------

######################################################################
# OGRE AUDIO-VIDEO BUILD SYSTEM
# Welcome to the CMake build system for OGRE AUDIO-VIDEO.
# This is the main file where we prepare the general build environment
# and provide build configuration options.
######################################################################

cmake_minimum_required(VERSION 3.0.0)
project(OgreVideoAudio)

find_package(OGRE REQUIRED OPTIONAL_COMPONENTS Bites CONFIG)

option(BUILD_DEMOS "Build demo (requires Ogre Bites and both Audio and Video)" ON)
option(BUILD_VIDEOPLUGIN "Build the Theora Video plugin" ON)
option(BUILD_AUDIOPLUGIN "Build the OggSound Audio plugin" ON)
include(GenerateExportHeader)

#####################################################################
# Set up the basic build environment
#####################################################################

if (NOT CMAKE_BUILD_TYPE)
  # CMake defaults to leaving CMAKE_BUILD_TYPE empty. This screws up differentiation between debug and release builds.
  set(CMAKE_BUILD_TYPE "RelWithDebInfo" CACHE STRING "Choose the type of build, options are: None (CMAKE_CXX_FLAGS or CMAKE_C_FLAGS used) Debug Release RelWithDebInfo MinSizeRel." FORCE)
endif ()

SET(CMAKE_DEBUG_POSTFIX "_d")
if(OGRE_PREFIX_DIR)
	set(CMAKE_INSTALL_PREFIX ${OGRE_PREFIX_DIR} CACHE PATH "plugin must end up in OGRE plugin path" FORCE)
endif()

if(BUILD_VIDEOPLUGIN)
	add_subdirectory(theoravideo)
endif()

if(BUILD_AUDIOPLUGIN)
	add_subdirectory(oggsound)
endif()

if (NOT OGRE_Bites_FOUND)
	if (BUILD_DEMOS)
		message(WARNING "NOT found OGRE Bites - disable build demos")
	endif (BUILD_DEMOS)
	set(BUILD_DEMOS OFF)
endif (NOT OGRE_Bites_FOUND)

if (NOT BUILD_VIDEOPLUGIN OR NOT BUILD_AUDIOPLUGIN)
	if (BUILD_DEMOS)
		message(WARNING "Building Demos require both audio and video plugins - disable build demos")
	endif (BUILD_DEMOS)
	set(BUILD_DEMOS OFF)
endif (NOT BUILD_VIDEOPLUGIN OR NOT BUILD_AUDIOPLUGIN)

if (BUILD_DEMOS)
	add_executable(player demos/player/player.cpp)
	target_link_libraries(player OgreBites Plugin_TheoraVideoSystem Plugin_OggSound)
	target_include_directories(player PUBLIC ${OPENAL_INCLUDE_DIRS})

	file(COPY demos/resources.cfg DESTINATION ${CMAKE_BINARY_DIR})
endif (BUILD_DEMOS)

# doxygen stuff
find_package(Doxygen)
if (DOXYGEN_FOUND)
  # prepare doxygen configuration file
  configure_file(${CMAKE_CURRENT_SOURCE_DIR}/scripts/Doxyfile.in ${CMAKE_CURRENT_BINARY_DIR}/Doxyfile)

  add_custom_target( docs
    COMMAND ${DOXYGEN_EXECUTABLE} ${CMAKE_CURRENT_BINARY_DIR}/Doxyfile
    WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
    COMMENT "Generating documentation with Doxygen."
    VERBATIM
  )

  # cleanup $build/api-doc on "make clean"
  set_property(DIRECTORY APPEND PROPERTY ADDITIONAL_MAKE_CLEAN_FILES docs)
endif (DOXYGEN_FOUND)
