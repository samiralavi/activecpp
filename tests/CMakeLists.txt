cmake_minimum_required(VERSION 3.1...3.14)

project(TestSuit CXX)

add_executable(${PROJECT_NAME} Application.cpp)
target_link_libraries(${PROJECT_NAME} samiralavi_activecpp::samiralavi_activecpp)

add_test(
  NAME ${PROJECT_NAME}
  COMMAND $<TARGET_FILE:${PROJECT_NAME}>
)
