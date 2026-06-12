# cmake file for building PandoraSDK
#-------------------------------------------------------------------------------------------------------------------------------------------
cmake_minimum_required(VERSION 2.8.2 FATAL_ERROR)

if(${CMAKE_SOURCE_DIR} STREQUAL ${CMAKE_BINARY_DIR})
    message(FATAL_ERROR "PandoraSDK requires an out-of-source build.")
endif()

# project name
project(PandoraSDK)

# project version
set(${PROJECT_NAME}_VERSION_MAJOR 03)
set(${PROJECT_NAME}_VERSION_MINOR 04)
set(${PROJECT_NAME}_VERSION_PATCH 01)
set(${PROJECT_NAME}_VERSION "${${PROJECT_NAME}_VERSION_MAJOR}.${${PROJECT_NAME}_VERSION_MINOR}.${${PROJECT_NAME}_VERSION_PATCH}")

#-------------------------------------------------------------------------------------------------------------------------------------------
# Dependencies
# include(PandoraCMakeSettings)  # external packaging macros; bypassed - we only need the .so

# Prefer local include directory to any paths to installed header files
include_directories(include)

#-------------------------------------------------------------------------------------------------------------------------------------------
# Low level settings - compiler etc
set(CMAKE_CXX_FLAGS "-O2 -g -Wall -Wextra -pedantic -Wno-long-long -Wno-sign-compare -fno-strict-aliasing -std=c++17 ${CMAKE_CXX_FLAGS}")

include(CheckCXXCompilerFlag)
unset(COMPILER_SUPPORTS_CXX_FLAGS CACHE)
CHECK_CXX_COMPILER_FLAG(${CMAKE_CXX_FLAGS} COMPILER_SUPPORTS_CXX_FLAGS)

if(NOT COMPILER_SUPPORTS_CXX_FLAGS)
    message(FATAL_ERROR "The compiler ${CMAKE_CXX_COMPILER} does not support cxx flags ${CMAKE_CXX_FLAGS}")
endif()

#-------------------------------------------------------------------------------------------------------------------------------------------
# Build products

# - Collect sources - not ideal because you have to keep running CMake to pick up changes
file(GLOB_RECURSE PANDORA_SDK_SRCS RELATIVE ${PROJECT_SOURCE_DIR} "src/*.cc")

# - Add library and properties
add_library(${PROJECT_NAME} SHARED ${PANDORA_SDK_SRCS})
set_target_properties(${PROJECT_NAME} PROPERTIES VERSION ${${PROJECT_NAME}_VERSION} SOVERSION ${${PROJECT_NAME}_VERSION_MAJOR}.${${PROJECT_NAME}_VERSION_MINOR})

# - Optional documents
option(PandoraSDK_BUILD_DOCS "Build documentation for ${PROJECT_NAME}" OFF)
if(PandoraSDK_BUILD_DOCS)
    add_subdirectory(doc)
endif()

#-------------------------------------------------------------------------------------------------------------------------------------------
# Install products

# - library
install(TARGETS ${PROJECT_NAME} DESTINATION lib COMPONENT Runtime)

# - headers
install(DIRECTORY include/ DESTINATION include COMPONENT Development FILES_MATCHING PATTERN "*.h")

# - minimal hand-rolled package config so find_package(PandoraSDK) resolves THIS install
# (the PandoraCMakeSettings packaging macros are bypassed; without a config file, consumers
# fall through to the key4hep/cvmfs PandoraSDKConfig.cmake and silently link the stock SDK).
file(GENERATE OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Config.cmake CONTENT "
SET( PandoraSDK_ROOT \"${CMAKE_INSTALL_PREFIX}\" )
SET( PandoraSDK_VERSION \"${${PROJECT_NAME}_VERSION}\" )
SET( PandoraSDK_INCLUDE_DIRS \"${CMAKE_INSTALL_PREFIX}/include\" )
SET( PandoraSDK_LIBRARY_DIRS \"${CMAKE_INSTALL_PREFIX}/lib\" )
FIND_LIBRARY( PandoraSDK_LIBRARIES NAMES PandoraSDK PATHS \"${CMAKE_INSTALL_PREFIX}/lib\" NO_DEFAULT_PATH )
SET( PandoraSDK_FOUND TRUE )
")
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Config.cmake DESTINATION .)
