cmake_minimum_required(VERSION 3.20)

project(impact LANGUAGES CXX)

add_executable(impact
	apps/main.cpp
)

target_compile_features(impact PRIVATE cxx_std_23)

target_include_directories(impact PRIVATE
	${PROJECT_SOURCE_DIR}/include
)

if(MSVC)
	target_compile_options(impact PRIVATE /W4 /permissive-)
else()
	target_compile_options(impact PRIVATE -Wall -Wextra -Wpedantic)
endif()
