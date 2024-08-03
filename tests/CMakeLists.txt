cmake_minimum_required(VERSION 3.1...3.14)

project(TestSuit CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

set(THREADS_PREFER_PTHREAD_FLAG ON)
find_package(Threads REQUIRED)

add_executable(${PROJECT_NAME} Application.cpp)
target_link_libraries(${PROJECT_NAME} samiralavi_activecpp::samiralavi_activecpp Threads::Threads)

add_test(
  NAME ${PROJECT_NAME}
  COMMAND $<TARGET_FILE:${PROJECT_NAME}>
)
