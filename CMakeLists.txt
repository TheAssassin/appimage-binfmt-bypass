cmake_minimum_required(VERSION 3.2)
project(appimage-binfmt-bypass)

set(CMAKE_C_STANDARD 11)
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_C_STANDARD_REQUIRED ON)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# configure names globally
set(bypass_bin binfmt-bypass)
set(preload_lib binfmt-bypass-preload)

# library to be preloaded when launching the patched runtime binary
# we need to build with -fPIC, otherwise we can't use it with $LD_PRELOAD
add_library(${preload_lib} SHARED preload.c logging.h)
target_compile_options(${preload_lib} PRIVATE -fPIC -DCOMPONENT_NAME="lib")
target_link_libraries(${preload_lib} PRIVATE dl)

# binary that extracts the runtime, patches it and launches it, preloading the library
add_executable(${bypass_bin} main.cpp elf.cpp logging.h elf.h)
target_link_libraries(${bypass_bin} PRIVATE dl)
target_compile_options(${bypass_bin} PRIVATE -DPRELOAD_LIB_NAME="./$<TARGET_FILE_NAME:${preload_lib}>" -DCOMPONENT_NAME="bin")

# the binary uses the library, so let's make sure it's up to date before we build the bypass binary
add_dependencies(${bypass_bin} ${preload_lib})
