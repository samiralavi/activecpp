cmake_minimum_required(VERSION 3.1...3.14)

project(HelloWorld CXX)

set(THREADS_PREFER_PTHREAD_FLAG ON)
find_package(Threads REQUIRED)

add_executable(${PROJECT_NAME} Application.cpp World.cpp)
target_link_libraries(${PROJECT_NAME} samiralavi_activecpp::samiralavi_activecpp Threads::Threads)
