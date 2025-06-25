cmake_minimum_required(VERSION 3.30)

list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_LIST_DIR}/cmake")

project(extents VERSION 0.1.0 DESCRIPTION "Simple and fast local extents managements" LANGUAGES CXX)
option(EXTENTS_BUILD_STATIC "Build static library" ON)
option(EXTENTS_WITH_CCACHE "Build with ccache" ON)
option(EXTENTS_WITH_MOLD "Build with mold" ON)
option(EXTENTS_SANITIZE "Build with address sanitizer" OFF)
option(EXTENTS_ENABLE_UBSAN "Build with undefined sanitizer when address sanitizer is enabled" OFF)

if (EXTENTS_WITH_CCACHE)
  find_program(CCACHE_FOUND ccache)
  if (CCACHE_FOUND)
    set_property(GLOBAL PROPERTY RULE_LAUNCH_COMPILE ccache)
    set_property(GLOBAL PROPERTY RULE_LAUNCH_LINK ccache)
  endif (CCACHE_FOUND)
endif ()

if (EXTENTS_WITH_MOLD)
  if (EXISTS "/usr/bin/mold")
    add_link_options(-fuse-ld=mold)
  endif ()
endif ()

if (EXTENTS_BUILD_STATIC)
  set (BUILD_SHARED_LIBS OFF CACHE BOOL "Build shared library")
endif ()

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

# https://github.com/llvm/llvm-project/issues/102443
if (CMAKE_CXX_COMPILER_ID STREQUAL "Clang")
  add_compile_options($<$<COMPILE_LANGUAGE:CXX>:-fclang-abi-compat=17>)
  add_compile_options($<$<COMPILE_LANGUAGE:CXX>:-fsized-deallocation>)
endif ()

if (APPLE)
  # for std::stop_source
  # https://libcxx.llvm.org/Status/Cxx20.html
  add_compile_options($<$<COMPILE_LANGUAGE:CXX>:-fexperimental-library>)
  add_compile_definitions(APPLE)
endif ()

if (EXTENTS_SANITIZE STREQUAL "Address")
  add_compile_options(-fsanitize=address)
  add_link_options(-fsanitize=address)
  if (EXTENTS_ENABLE_UBSAN)
    add_compile_options(-fsanitize=undefined)
    add_link_options(-fsanitize=undefined)
  endif ()
elseif (EXTENTS_SANITIZE STREQUAL "Thread")
  add_compile_options(-fsanitize=thread)
  add_link_options(-fsanitize=thread)
endif ()

# sources
add_subdirectory(lib)
