# Using Google Test with cmake

## Table of contents

1. [Foreword](#foreword)
1. [What is what](#what-is-what)
1. [Configuring `cmake` to download and generate build files for `gtest` and your project](#configuring-cmake-to-download-and-generate-build-files-for-gtest-and-your-project)
1. [Building your project](#building-your-project)

## Foreword

My aim with this guide is to provide an easy to follow step-by-step tutorial for setting up `gtest` with `cmake` as the build tool.

I wrote this guide mainly for the users of the [CLion](https://www.jetbrains.com/clion/) IDE but at the end I included some CLI commands to build your project from the terminal. <small>(You may find these imperfect as these are not the main focus of this guide.)</small>

If you find any mistakes or errors in this guide feel free to open an *Issue* on [Github](https://github.com/marcellBan/cmake-gtest) or even fork my repository, fix the issue and open a *Pull request*.

## What is what

### *Google Test*

`gtest` is a free, open-source *C++* unit testing framework developed mainly by *Google* and is available on [Github](https://github.com/google/googletest). In this guide we will set it up for use with the `cmake` build tool. It will be refered to as `gtest` or *Google Test*.

### *CMake*

From the *CMake* website:
> CMake is an open-source, cross-platform family of tools designed to build, test and package software.

We will use this build tool to automate the integration of `gtest` into our project. If you use some other build tool then I'm afraid that you'll have to find some other tutorial to help you as mine is geared directly torwards *CMake*.

## Configuring `cmake` to download and generate build files for `gtest` and your project

<small>Note that the main parts of the files in this section are taken from the official [*Google Test*](https://github.com/google/googletest) git repository.</small>

First we will need to define a 'side project' in a separate file (note that this file could be named differently but you should take care that it is accessed properly from your projects main `CMakeLists.txt`):

`CMakeLists.txt.in`

```cmake
cmake_minimum_required(VERSION 2.8.2)

project(googletest-download NONE)

include(ExternalProject)
ExternalProject_Add(googletest
        GIT_REPOSITORY https://github.com/google/googletest.git
        GIT_TAG master
        SOURCE_DIR "${CMAKE_BINARY_DIR}/googletest-src"
        BINARY_DIR "${CMAKE_BINARY_DIR}/googletest-build"
        CONFIGURE_COMMAND ""
        BUILD_COMMAND ""
        INSTALL_COMMAND ""
        TEST_COMMAND ""
        )
```

The `CMakeLists.txt` file should look something like this:

```cmake
cmake_minimum_required(VERSION 2.8.2)
project(tutorial)
set(CMAKE_CXX_STANDARD 11) # use the c++11 standard

# Download and unpack googletest at configure time
configure_file(CMakeLists.txt.in googletest-download/CMakeLists.txt)
execute_process(COMMAND ${CMAKE_COMMAND} -G "${CMAKE_GENERATOR}" .
        RESULT_VARIABLE result
        WORKING_DIRECTORY ${CMAKE_BINARY_DIR}/googletest-download)
if (result)
    message(FATAL_ERROR "CMake step for googletest failed: ${result}")
endif ()
execute_process(COMMAND ${CMAKE_COMMAND} --build .
        RESULT_VARIABLE result
        WORKING_DIRECTORY ${CMAKE_BINARY_DIR}/googletest-download)
if (result)
    message(FATAL_ERROR "Build step for googletest failed: ${result}")
endif ()

# Prevent overriding the parent project's compiler/linker
# settings on Windows
set(gtest_force_shared_crt ON CACHE BOOL "" FORCE)

# Add googletest directly to our build. This defines
# the gtest and gtest_main targets.
add_subdirectory(${CMAKE_BINARY_DIR}/googletest-src
        ${CMAKE_BINARY_DIR}/googletest-build)

# The gtest/gtest_main targets carry header search path
# dependencies automatically when using CMake 2.8.11 or
# later. Otherwise we have to add them here ourselves.
if (CMAKE_VERSION VERSION_LESS 2.8.11)
    include_directories("${gtest_SOURCE_DIR}/include")
endif ()


# google test unit tests
set(TEST_SOURCES tutorial.cpp lib.h lib.cpp)
add_executable(tutorialTest ${TEST_SOURCES})
target_link_libraries(tutorialTest gtest_main)

# release build
set(RELEASE_SOURCES lib.h lib.cpp)
add_library(tutorialLib ${RELEASE_SOURCES})
```

The above configuration assumes the following:

- your unit tests and the [standard `main` function](https://github.com/google/googletest/blob/master/googletest/docs/Primer.md#writing-the-main-function) is located in the `tutorial.cpp` file
- your code under test is contained in `lib.h` and `lib.cpp` (you can extend the `TEST_SOURCES` variable as needed <small>the CLion IDE will offer some help with that</small>)
- for the release you just want a library not an executable (of course you can change this to `add_executable()` if your code has a separate `main` function in some other file <small>change the `RELEASE_SOURCES` variable accordingly</small>)

Furthermore you can change any names listed below to your liking:

- `CMakeLists.txt.in`
- `TEST_SOURCES`
- `RELEASE_SOURCES`
- `tutorialTest`
- `tutorialLib`

## Building your project

If you are using *CLion* it will detect the changes in the *CMake* project and offer to reload it (or reload it automatically). Then you can select the target `tutorialTest` or `tutorialLib` to build and run the proper config. <small>(Note that you will see some other targets added by the *Google Test* framework, you don't need to build or run these as they are automatically built as dependencies when needed.)</small>

From the command line you can run `cmake` on the directory of your project to generate the build files and the run `make` to build the project. <small>(There are a lot of configuration options here and the above commands may not work without some more tinkering but this is not the guide to solve those issues.)</small>
