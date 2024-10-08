# Copyright (c) Microsoft Corporation.
# Licensed under the MIT license.

cmake_minimum_required(VERSION 3.17.1)
project(mapi-scrubbed)

set(CMAKE_CXX_STANDARD 20)

# Fetch the https://github.com/microsoft/MAPIStubLibrary repository.
include(FetchContent)
FetchContent_Declare(MAPIStubLibrary
  GIT_REPOSITORY "https://github.com/microsoft/MAPIStubLibrary"
  GIT_TAG "1d30c31ebf05ef444371520cd4268d6e1fda8a3b"
  GIT_PROGRESS TRUE)
FetchContent_GetProperties(MAPIStubLibrary)

if(NOT mapistublibrary_POPULATED)
  FetchContent_Populate(MAPIStubLibrary)
endif()

add_subdirectory(scrubber ${mapistublibrary_BINARY_DIR})

file(GLOB MAPI_HEADERS LIST_DIRECTORIES false "${mapistublibrary_SOURCE_DIR}/include/*.h")
add_custom_command(OUTPUT copied_headers
	COMMAND cmake -E make_directory ${CMAKE_BINARY_DIR}/include
	COMMAND cmake -E copy_if_different ${MAPI_HEADERS} ${mapistublibrary_SOURCE_DIR}/include/MAPIConstants.h ${CMAKE_BINARY_DIR}/include
	COMMAND cmake -E touch copied_headers
	DEPENDS scrubconstants)
add_custom_target(copyheaders ALL DEPENDS copied_headers)

add_library(testbuild STATIC testbuild.cpp)
add_dependencies(testbuild copyheaders)
target_include_directories(testbuild PUBLIC ${CMAKE_BINARY_DIR}/include)

add_subdirectory(importlib)

install(FILES
    ${MAPI_HEADERS}
    ${mapistublibrary_SOURCE_DIR}/include/MAPIConstants.h
  DESTINATION include)
