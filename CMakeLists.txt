cmake_minimum_required(VERSION 3.16)

# set the project name
project(activecpp VERSION 0.1
        DESCRIPTION "Active Objects in CPP."
        LANGUAGES CXX)

# specify the C++ standard
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED True)


add_library(ACTIVECPP INTERFACE)
target_include_directories(ACTIVECPP INTERFACE external/activecpp/include)
