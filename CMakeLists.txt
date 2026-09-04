cmake_minimum_required(VERSION 3.20)

# ==========================================
# Project Configuration
# ==========================================
project(impact LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 23)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# ==========================================
# FetchContent Module
# ==========================================
include(FetchContent)

# ==========================================
# 1. Fetch raylib
# ==========================================
# Turn off raylib examples/tests to speed up your build
set(BUILD_EXAMPLES OFF CACHE INTERNAL "")

FetchContent_Declare(
    raylib
    GIT_REPOSITORY https://github.com
    GIT_TAG        5.0
)
FetchContent_MakeAvailable(raylib)

# ==========================================
# 2. Fetch googletest
# ==========================================
# Prevent overriding compiler settings on Windows
set(gtest_force_shared_crt ON CACHE BOOL "" FORCE)

FetchContent_Declare(
    googletest
    GIT_REPOSITORY https://github.com
    GIT_TAG        v1.14.0 
)
FetchContent_MakeAvailable(googletest)

# ==========================================
# 3. Main Application Target
# ==========================================
add_executable(impact
	apps/main.cpp
)

target_include_directories(impact PRIVATE
	${PROJECT_SOURCE_DIR}/include
)

target_link_libraries(impact PRIVATE raylib)

# ==========================================
# 4. Testing Target
# ==========================================
# Enable CTest framework
enable_testing()
add_subdirectory(tests)

# ==========================================
# 5. Compiler Options
# ==========================================
if(MSVC)
	target_compile_options(impact PRIVATE /W4 /permissive-)
else()
	target_compile_options(impact PRIVATE -Wall -Wextra -Werror -Wpedantic)
endif()