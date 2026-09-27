cmake_minimum_required(VERSION 3.31)

project(yamc C CXX)

# Requires C++11
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

option(ENABLE_TESTING "Enable unit testing." ON)

add_library(yamc INTERFACE)
add_library(yamc::yamc ALIAS yamc)

target_include_directories(yamc INTERFACE "${CMAKE_CURRENT_LIST_DIR/include")

if (ENABLE_TESTING)
  enable_testing()
  add_subdirectory("${CMAKE_CURRENT_LIST_DIR/tests")
endif()

install(DIRECTORY "${CMAKE_CURRENT_LIST_DIR/include" DESTINATION "include/yamc"
    FILES_MATCHING PATTERN "*.hpp")
