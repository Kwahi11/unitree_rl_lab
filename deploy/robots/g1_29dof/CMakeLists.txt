cmake_minimum_required(VERSION 3.12)
project(g1_29dof_controller)

set(CMAKE_CXX_STANDARD 17)

set(UNITREE_SDK2_DEFAULT_PREFIX "/opt/unitree_robotics")

if(NOT unitree_sdk2_DIR)
  if(DEFINED ENV{UNITREE_SDK2_DIR} AND EXISTS "$ENV{UNITREE_SDK2_DIR}/unitree_sdk2Config.cmake")
    set(unitree_sdk2_DIR "$ENV{UNITREE_SDK2_DIR}" CACHE PATH "Path to unitree_sdk2 CMake config")
  elseif(EXISTS "/home/xxx/worksapce/unitree_sdk2/lib/cmake/unitree_sdk2/unitree_sdk2Config.cmake")
    set(unitree_sdk2_DIR "/home/xxx/worksapce/unitree_sdk2/lib/cmake/unitree_sdk2" CACHE PATH "Path to unitree_sdk2 CMake config")
  elseif(EXISTS "${UNITREE_SDK2_DEFAULT_PREFIX}/lib/cmake/unitree_sdk2/unitree_sdk2Config.cmake")
    set(unitree_sdk2_DIR "${UNITREE_SDK2_DEFAULT_PREFIX}/lib/cmake/unitree_sdk2" CACHE PATH "Path to unitree_sdk2 CMake config")
  endif()
endif()

if(unitree_sdk2_DIR)
  find_package(unitree_sdk2 REQUIRED CONFIG PATHS "${unitree_sdk2_DIR}" NO_DEFAULT_PATH)
else()
  find_package(unitree_sdk2 REQUIRED CONFIG)
endif()

find_package(Boost REQUIRED COMPONENTS program_options)
find_package(yaml-cpp REQUIRED)
find_package(Eigen3 REQUIRED)

set(ONNXRUNTIME_LIB ${PROJECT_SOURCE_DIR}/../../thirdparty/onnxruntime-linux-x64-1.22.0/lib/libonnxruntime.so.1.22.0)

file(GLOB_RECURSE ADD_SRC_LIST
  ${PROJECT_SOURCE_DIR}/src/*.cpp
)

add_library(${PROJECT_NAME}_lib ${ADD_SRC_LIST})

target_include_directories(${PROJECT_NAME}_lib PUBLIC
  include
  ${PROJECT_SOURCE_DIR}/../../thirdparty/onnxruntime-linux-x64-1.22.0/include
  ${PROJECT_SOURCE_DIR}/../../include/
  ${EIGEN3_INCLUDE_DIRS}
)

target_link_libraries(${PROJECT_NAME}_lib PRIVATE
  Boost::program_options
  yaml-cpp
  fmt
  rt
  pthread
  ${ONNXRUNTIME_LIB}
)

target_link_libraries(${PROJECT_NAME}_lib PUBLIC
  unitree_sdk2
)

add_executable(g1_ctrl main.cpp)
target_link_libraries(g1_ctrl PRIVATE ${PROJECT_NAME}_lib)