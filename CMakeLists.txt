cmake_minimum_required(VERSION 3.5)
project(breads_engine VERSION 0.1.0)

add_executable(breads_engine src/config.h src/main.cpp)

target_include_directories(breads_engine PRIVATE libs)

find_package(OpenGL REQUIRED)

target_link_libraries(breads_engine PRIVATE glfw3 ${OPENGL_gl_LIBRARY})