cmake_minimum_required(VERSION 3.20 FATAL_ERROR)

# Disable QtCreator generating a conan provider file
set(QT_CREATOR_SKIP_PACKAGE_MANAGER_SETUP ON)

# Conan 2 support
set(CMAKE_PROJECT_TOP_LEVEL_INCLUDES "${CMAKE_SOURCE_DIR}/conan_provider.cmake")

project(cmake-conan
    VERSION 0.1.0
    DESCRIPTION "CMake with Conan 2 example"
    HOMEPAGE_URL "https://github.com/mathausmendel/cmake-conan"
    LANGUAGES CXX C)

# Define the available build types and set it to Debug it none is selected
if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
    set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS "Debug" "Release" "MinSizeRel" "RelWithDebInfo")
    
    set(DEFAULT_CMAKE_BUILD_TYPE Debug)
    set(CMAKE_BUILD_TYPE "${DEFAULT_CMAKE_BUILD_TYPE}" CACHE STRING "Choose the type of build." FORCE)
    message(STATUS "Setting build type to '${DEFAULT_CMAKE_BUILD_TYPE}' as none was specified.")
endif()

# Set default output paths
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/lib")
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/lib")
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/bin")

# Disable developer warnings for CMake
set(CMAKE_SUPPRESS_DEVELOPER_WARNINGS 1 CACHE INTERNAL "No dev warnings")

# Export all the compile commands
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

# Use ccache if available
find_program(CCACHE_FOUND ccache HINTS /bin /usr/bin)
if(CCACHE_FOUND)
        set_property(GLOBAL PROPERTY RULE_LAUNCH_COMPILE ccache)
        set_property(GLOBAL PROPERTY RULE_LAUNCH_LINK ccache)
endif()

# Set C++ version to 20
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Enable PIC
set(CMAKE_POSITION_INDEPENDENT_CODE TRUE)

# Disable exporting all symbols by default
set(CMAKE_VISIBILITY_INLINES_HIDDEN TRUE)
set(CMAKE_CXX_VISIBILITY_PRESET hidden)
set(CMAKE_C_VISIBILITY_PRESET hidden)

# Compiler flags
if("${CMAKE_CXX_COMPILER_ID}" MATCHES "GNU")
    add_compile_options(-Wall
        -Wextra
        -Wpedantic
        -flto=auto
        -fstack-protector-strong
        -fstack-clash-protection
        -fdiagnostics-color=auto
        -grecord-gcc-switches
        -g3
        -pipe)

    add_definitions(-D_GLIBCXX_ASSERTIONS)

    if(CMAKE_CXX_COMPILER_VERSION VERSION_GREATER 12)
        add_compile_options(-ftrivial-auto-var-init=zero)

        # First we need to undef _FORTIFY_SOURCE as some systems
        # or libraries might have it set it previously.
        set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -U_FORTIFY_SOURCE -D_FORTIFY_SOURCE=3 ")
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -U_FORTIFY_SOURCE -D_FORTIFY_SOURCE=3 ")
    else()
        add_definitions(-D_FORTIFY_SOURCE=2)
    endif()

    add_link_options(-Wl,-z,noexecstack
        -Wl,-z,relro
        -Wl,-z,now
        -Wl,--no-as-needed
        -Wl,--gc-sections
        -Wl,--discard-all)
endif()

# Find the required packages
find_package(Boost REQUIRED)

add_subdirectory(src)
