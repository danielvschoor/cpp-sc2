cmake_minimum_required(VERSION 3.15)

project(cpp-sc2)

option(BUILD_SC2_RENDERER "Build SC2 Renderer library" ON)
option(BUILD_API_EXAMPLES "Build Examples" ON)
option(BUILD_API_TESTS "Build Tests" ON)

set(SC2_VERSION "5.0.10" CACHE STRING "Version of the target StarCraft II client")
message("Target SC2 version: ${SC2_VERSION}")

# Use bin as the directory for all executables.
# This will make protoc easy to find.
set(EXECUTABLE_OUTPUT_PATH ${PROJECT_BINARY_DIR}/bin)
set(LIBRARY_OUTPUT_PATH ${PROJECT_BINARY_DIR}/bin)

# Windows builds subdirectories Debug/Release.
# These variables will overwrite that and put binaries in bin.
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY_DEBUG ${PROJECT_BINARY_DIR}/bin)
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY_DEBUG ${PROJECT_BINARY_DIR}/bin)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY_DEBUG ${PROJECT_BINARY_DIR}/bin)

set(CMAKE_LIBRARY_OUTPUT_DIRECTORY_RELEASE ${PROJECT_BINARY_DIR}/bin)
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY_RELEASE ${PROJECT_BINARY_DIR}/bin)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY_RELEASE ${PROJECT_BINARY_DIR}/bin)

set(CMAKE_LIBRARY_OUTPUT_DIRECTORY_MINSIZEREL ${PROJECT_BINARY_DIR}/bin)
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY_MINSIZEREL ${PROJECT_BINARY_DIR}/bin)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY_MINSIZEREL ${PROJECT_BINARY_DIR}/bin)

set(CMAKE_LIBRARY_OUTPUT_DIRECTORY_RELWITHDEBINFO ${PROJECT_BINARY_DIR}/bin)
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY_RELWITHDEBINFO ${PROJECT_BINARY_DIR}/bin)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY_RELWITHDEBINFO ${PROJECT_BINARY_DIR}/bin)

# Build with c++14 support.
set(CMAKE_CXX_STANDARD 14)

# Use statically linked runtime
set(CMAKE_MSVC_RUNTIME_LIBRARY "MultiThreaded$<$<CONFIG:Debug>:Debug>")

# Allow creating filters for projects in visual studio.
set_property(GLOBAL PROPERTY USE_FOLDERS ON)

# -- Settings for building civetweb
# Don't build civetweb tests.
set(CIVETWEB_BUILD_TESTING OFF CACHE BOOL "" FORCE)
set(CIVETWEB_ENABLE_WEBSOCKETS ON CACHE BOOL "" FORCE)
set(CIVETWEB_ENABLE_SERVER_EXECUTABLE OFF CACHE BOOL "" FORCE)
set(CIVETWEB_ENABLE_IPV6 ON CACHE BOOL "" FORCE)

# Don't build civetweb with sanitizers
set(CIVETWEB_ENABLE_ASAN OFF CACHE BOOL "" FORCE)

# Don't build protobuf tests.
set(protobuf_BUILD_TESTS OFF CACHE BOOL "" FORCE)
set(protobuf_BUILD_EXAMPLES OFF CACHE BOOL "" FORCE)

# Don't build SDL dynamic lib.
set(SDL_SHARED OFF CACHE BOOL "" FORCE)

# Run civetwebs cmake.
add_subdirectory("contrib/civetweb")

# Enable IPv6 in civetweb
target_compile_options(civetweb-c-library PUBLIC -DUSE_IPV6=1)

# Run protobufs cmake.
add_subdirectory("contrib/protobuf/cmake")

set_target_properties(civetweb-c-library PROPERTIES FOLDER contrib)
set_target_properties(libprotobuf PROPERTIES FOLDER contrib)
set_target_properties(libprotobuf-lite PROPERTIES FOLDER contrib)
set_target_properties(libprotoc PROPERTIES FOLDER contrib)
set_target_properties(protoc PROPERTIES FOLDER contrib)

# civetweb's cmake does not set debug postfix
set_target_properties(civetweb-c-library PROPERTIES DEBUG_POSTFIX "d")

if (MSVC)
    # Setup MSVC parallelized builds
    add_compile_options($<$<CXX_COMPILER_ID:MSVC>:/MP>)

    set_target_properties(libprotobuf libprotobuf-lite libprotoc protoc PROPERTIES COMPILE_FLAGS "/W0")
    set_source_files_properties(${libprotobuf_files} PROPERTIES COMPILE_FLAGS "/W0")
    set_source_files_properties(${protobuf_SHARED_OR_STATIC} PROPERTIES COMPILE_FLAGS "/W0")
    set_source_files_properties(${libprotobuf_lite_files} PROPERTIES COMPILE_FLAGS "/W0")
    add_definitions(-D_SCL_SECURE_NO_WARNINGS)
endif ()

add_subdirectory("src")

if (BUILD_SC2_RENDERER)
    add_definitions(-DBUILD_SC2_RENDERER)

    add_subdirectory("contrib/SDL")

    set_target_properties(SDL2main PROPERTIES FOLDER contrib)
    set_target_properties(SDL2-static PROPERTIES FOLDER contrib)

    set_target_properties(sc2renderer PROPERTIES FOLDER utilities)
    set_target_properties(uninstall PROPERTIES FOLDER CMakePredefinedTargets)

    target_compile_definitions(SDL2-static PRIVATE -DHAVE_LIBC)
endif ()

if (BUILD_API_EXAMPLES)
    add_subdirectory("examples")
endif ()

if (BUILD_API_TESTS)
    add_subdirectory("tests")
endif ()
