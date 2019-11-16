cmake_minimum_required(VERSION 3.13 FATAL_ERROR)
project("NanoVis" LANGUAGES C CXX)

list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_LIST_DIR}/cmake/Modules")

# ccache
find_program(CCACHE_FOUND ccache)
if(CCACHE_FOUND)
    message(STATUS "Found CCache")
    set_property(GLOBAL PROPERTY RULE_LAUNCH_COMPILE ccache)
    set_property(GLOBAL PROPERTY RULE_LAUNCH_LINK ccache)
endif(CCACHE_FOUND)

include(CMakeDependentOption)
# superbuild settings
include(SuperBuildDepends)

# cpp standard settings
superbuild_option(cpp17)
superbuild_option(modern-cpp)

# dependencies settings
superbuild_depend(eigen)
superbuild_depend(glfw)
superbuild_depend(nanogui)
superbuild_extern(opencv)

# always generate PIC code, no matter static or shared library
set(CMAKE_POSITION_INDEPENDENT_CODE ON)

# project directories settings
set(NANOVIS_DIR ${CMAKE_CURRENT_SOURCE_DIR})
set(NANOVIS_INCLUDE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/include)
set(NANOVIS_SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/src)
# set(NANOVIS_BINARY_INCLUDE_DIR ${CMAKE_CURRENT_BINARY_DIR}/include)
# set(NANOVIS_TEST_SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/test)

# some build options
option(NANOVIS_BUILD_EXAMPLE "Build nanovis example application" ON)

set(NANOVIS_PUBLIC_HEADERS
    ${NANOVIS_INCLUDE_DIR}/nanovis/nanovis.h
)

set(NANOVIS_PUBLIC_SOURCES
)

set(NANOVIS_PRIVATE_HEADERS
    ${NANOVIS_SOURCE_DIR}/screen.h
    ${NANOVIS_SOURCE_DIR}/window.h
)

set(NANOVIS_PRIVATE_SOURCES
    ${NANOVIS_SOURCE_DIR}/nanovis.cpp
    ${NANOVIS_SOURCE_DIR}/screen.cpp
    ${NANOVIS_SOURCE_DIR}/window.cpp
)

add_library(nanovis)
target_compile_definitions(nanovis
    PRIVATE -DGL_SILENCE_DEPRECATION
)
if(CMAKE_CXX_COMPILER_ID MATCHES "Clang")
    target_compile_options(nanovis
        PRIVATE
            -Qunused-arguments
            -mllvm
            -inline-threshold=5000
    )
endif()
if(MSVC)
    target_compile_definitions(nanovis
        PRIVATE
            _CRT_SECURE_NO_WARNINGS
            _USE_MATH_DEFINES
    )
endif()
set_target_properties(nanovis PROPERTIES CXX_EXTENSIONS OFF) # -std=c++17 instead of -std=g++17
target_sources(nanovis
    PRIVATE
        ${NANOVIS_PUBLIC_HEADERS}
        ${NANOVIS_PUBLIC_SOURCES}
        ${NANOVIS_PRIVATE_HEADERS}
        ${NANOVIS_PRIVATE_SOURCES}
)
target_include_directories(nanovis
    PUBLIC
        ${NANOVIS_INCLUDE_DIR}
        # ${NANOVIS_BINARY_INCLUDE_DIR}
    PRIVATE
        ${NANOVIS_SOURCE_DIR}
)
target_link_libraries(nanovis
    PUBLIC
        options::modern-cpp
        depends::eigen
        depends::glfw
        depends::nanogui
        depends::opencv
)

set(NANOVIS_LIBRARY_DIR ${CMAKE_CURRENT_BINARY_DIR} CACHE INTERNAL "")


# Build example application if desired
if(NANOVIS_BUILD_EXAMPLE)
    add_executable(example utils/example.cpp)
    target_link_libraries(example PRIVATE nanovis)
endif()
