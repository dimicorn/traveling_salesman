cmake_minimum_required(VERSION 4.0)
project(travelling_salesman)
set(CMAKE_CXX_STANDARD 20)

add_executable(random random_generator.cpp)
add_executable(main main.cpp)