# CMakeLists.txt
cmake_minimum_required(VERSION 2.8)
project(imu_tk)

# Set policies
cmake_policy(SET CMP0015 NEW)

# Set C++ standard to C++17
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Set build type
if ("${CMAKE_BUILD_TYPE}" STREQUAL "")
  set(CMAKE_BUILD_TYPE "Release")
endif()

# Set default value for BUILD_IMU_TK_EXAMPLES
if(NOT DEFINED BUILD_IMU_TK_EXAMPLES)
  set(BUILD_IMU_TK_EXAMPLES "ON")
endif()

# Find required packages
find_package(Boost REQUIRED)
find_package(Eigen3 REQUIRED)
find_package(Ceres REQUIRED)

# Include directories
include_directories(./include
                    /usr/include
                    ${Boost_INCLUDE_DIRS}
                    ${EIGEN3_INCLUDE_DIR}
                    ${CERES_INCLUDE_DIRS})

# Fallback for Eigen include directory
if(NOT EIGEN3_INCLUDE_DIR)
  include_directories("/usr/include/eigen3")
endif()

# Additional packages for visualization
find_package(Qt4 COMPONENTS QtCore QtGui QtOpenGL REQUIRED)
find_package(OpenGL REQUIRED)
find_package(GLUT REQUIRED)

include_directories(${OPENGL_INCLUDE_DIRS}
                    ${GLUT_INCLUDE_DIRS}
                    ${CMAKE_CURRENT_BINARY_DIR})

# Source files for visualization
set(IMU_TK_VIS_EXTRA_SOURCES 
    ./src/vis_extra/gl_camera.cpp 
    ./src/vis_extra/opengl_3d_scene.cpp)
set(IMU_TK_VIS_EXTRA_HEADERS
    ./include/imu_tk/vis_extra/opengl_3d_scene.h)

qt4_wrap_cpp(IMU_TK_VIS_EXTRA_HEADERS_MOC ${IMU_TK_VIS_EXTRA_HEADERS})

include(${QT_USE_FILE})
add_definitions(${QT_DEFINITIONS})

# Core source files
aux_source_directory(./src IMU_TK_CORE_SRC)
set(IMU_TK_SRC ${IMU_TK_CORE_SRC} ${IMU_TK_VIS_EXTRA_HEADERS_MOC} ${IMU_TK_VIS_EXTRA_SOURCES})

# Create library
add_library(imu_tk ${IMU_TK_SRC})
set_target_properties(imu_tk PROPERTIES ARCHIVE_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/lib)

# Set include and library directories
set(IMU_TK_INCLUDE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/include CACHE STRING "imu_tk include directories")
set(IMU_TK_LIB_DIR ${CMAKE_CURRENT_SOURCE_DIR}/lib CACHE STRING "imu_tk libraries directories")
set(IMU_TK_LIBS imu_tk ${CERES_LIBRARIES} ${QT_LIBRARIES} ${OPENGL_LIBRARIES} ${GLUT_LIBRARY} 
     CACHE STRING "imu_tk libraries")

message("${IMU_TK_LIBS}")

# Build examples if enabled
if(BUILD_IMU_TK_EXAMPLES)
  add_executable(test_imu_calib apps/test_imu_calib.cpp)
  target_link_libraries(test_imu_calib ${IMU_TK_LIBS})
  set_target_properties(test_imu_calib PROPERTIES RUNTIME_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/bin)

  add_executable(test_integration apps/test_integration.cpp)
  target_link_libraries(test_integration ${IMU_TK_LIBS})
  set_target_properties(test_integration PROPERTIES RUNTIME_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/bin)
endif()

# Add cross-compilation checks (optional)
if(CMAKE_CROSSCOMPILING)
  message(STATUS "Cross-compiling for ${CMAKE_SYSTEM_PROCESSOR}")
endif()