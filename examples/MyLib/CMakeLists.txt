cmake_minimum_required(VERSION 3.1...3.14)

project(MyLib CXX)

add_library(${PROJECT_NAME} STATIC MyLib.cpp)
target_link_libraries(${PROJECT_NAME} samiralavi_activecpp::samiralavi_activecpp)
target_include_directories(${PROJECT_NAME} PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/include)
