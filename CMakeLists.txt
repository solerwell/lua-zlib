# Copyright (C) 2007-2009 LuaDist.
# Submitted by David Manura
# Redistribution and use of this file is allowed according to the
# terms of the MIT license.
# For details see the COPYRIGHT file distributed with LuaDist.
# Please note that the package source code is licensed under its own
# license.

# PROJECT(lua-zlib C)
CMAKE_MINIMUM_REQUIRED (VERSION 2.6)

message("Input--> $RESTY_DIR = ${RESTY_DIR}")

option(USE_LUA "Use Lua (also called 'C' Lua) includes (default)" OFF)
option(USE_LUAJIT "Use LuaJIT includes instead of 'C' Lua ones (recommended, if you're using LuaJIT, but disabled by default)" ON)
# set openresty root directory --> luajit & lua_lib directory
set(RESTY_DIR "/opt/openresty" CACHE STRING "The root directory of openresty installed, it doesn't end in a slash")
set(LUA_INCLUDE_DIR "${RESTY_DIR}/luajit/include/luajit-2.1" CACHE STRING "")
set(LUA_LIBRARIES  "${RESTY_DIR}/luajit/lib" CACHE STRING "")

set(USE_LUA_VERSION 5.1 CACHE STRING "Set the Lua version to use (default: 5.1)")

message("$USE_LUA = ${USE_LUA}")
message("$USE_LUAJIT = ${USE_LUAJIT}")
message("$LUA_INCLUDE_DIR = ${LUA_INCLUDE_DIR}")
message("$LUA_LIBRARIES = ${LUA_LIBRARIES}")

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/cmake/Modules/")

if(USE_LUAJIT)
        message("Using luajit, the system will try to find the package")
# Find luajit
        find_package(LuaJIT REQUIRED)
        set(USE_LUA OFF)
# / Find lua
endif()

if(USE_LUA)
        message("Using Clua, the system will try to find the package")
# Find lua
        find_package(Lua ${USE_LUA_VERSION} EXACT REQUIRED)
# / Find lua
endif()


# Basic configurations
  SET(INSTALL_CMOD share/lua/cmod CACHE PATH "Directory to install Lua binary modules (configure lua via LUA_CPATH)")
# / configs

# Find zlib
  FIND_PACKAGE(ZLIB REQUIRED)
# / Find zlib

# Define how to build zlib.so:
  INCLUDE_DIRECTORIES(${ZLIB_INCLUDE_DIRS} ${LUA_INCLUDE_DIR})
  ADD_LIBRARY(cmod_zlib MODULE
    lua_zlib.c zlib.def)
  SET_TARGET_PROPERTIES(cmod_zlib PROPERTIES PREFIX "")
  SET_TARGET_PROPERTIES(cmod_zlib PROPERTIES OUTPUT_NAME zlib)
  TARGET_LINK_LIBRARIES(cmod_zlib ${ZLIB_LIBRARIES})
# / build zlib.so

# Define how to test zlib.so:
  INCLUDE(CTest)
  SET(LUA_BIN "lua${LUA_VERSION_MAJOR}.${LUA_VERSION_MINOR}")
  FIND_PROGRAM(LUA NAMES ${LUA_BIN} lua luajit lua.bat)
  ADD_TEST(basic ${LUA} ${CMAKE_CURRENT_SOURCE_DIR}/test.lua ${CMAKE_CURRENT_SOURCE_DIR}/ ${CMAKE_CURRENT_BINARY_DIR}/)
  SET_TESTS_PROPERTIES(basic
                       PROPERTIES
                       FAIL_REGULAR_EXPRESSION
                       "not ok")
# / test zlib.so

# Where to install stuff
  INSTALL (TARGETS cmod_zlib DESTINATION ${INSTALL_CMOD})
# / Where to install.
