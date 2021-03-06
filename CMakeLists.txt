cmake_minimum_required(VERSION 3.1)
project(ORB_SLAM2)

if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE Release)
endif()

message("Build type: " ${CMAKE_BUILD_TYPE})

#Make using OpenGL optional
option(USE_VIEWER "Build with OpenGL support and MapViewer" ON)
if(USE_VIEWER)
  add_definitions(-DUSE_VIEWER)
  message("Building ORB_SLAM2 with Pangolin")
else()
  message("Building ORB_SLAM2 WITHOUT Pangolin")
endif()

#Make using CUDA bindings optional
option(USE_CUDA "Build with CUDA backend" OFF)
if(USE_CUDA)
  add_definitions(-DUSE_CUDA)
  message("Building ORB_SLAM2 with CUDA")
else()
  message("Building ORB_SLAM2 WITHOUT CUDA")
endif()

if(UNIX)
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS}  -Wall  -O3 -march=native ")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall   -O3 -march=native")

    # Check C++11 or C++0x support
    include(CheckCXXCompilerFlag)

    CHECK_CXX_COMPILER_FLAG("-std=c++11" COMPILER_SUPPORTS_CXX11)
    CHECK_CXX_COMPILER_FLAG("-std=c++0x" COMPILER_SUPPORTS_CXX0X)
    if(COMPILER_SUPPORTS_CXX11)
       set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
       add_definitions(-DCOMPILEDWITHC11)
       message(STATUS "Using flag -std=c++11.")
    elseif(COMPILER_SUPPORTS_CXX0X)
       set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++0x")
       add_definitions(-DCOMPILEDWITHC0X)
       message(STATUS "Using flag -std=c++0x.")
    else()
       message(FATAL_ERROR "The compiler ${CMAKE_CXX_COMPILER} has no C++11 support. Please use a different C++ compiler.")
    endif()
else()
    set(CMAKE_CXX_STANDARD 11)
    set(CMAKE_CXX_STANDARD_REQUIRED ON)
    add_definitions(-DCOMPILEDWITHC11)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /bigobj")
endif()

list(APPEND CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/cmake_modules)

if(USE_VIEWER)
  find_package(Pangolin REQUIRED)
endif(USE_VIEWER)

find_package(Eigen3 3.1.0 REQUIRED)

# DBoW2 target
add_subdirectory(${PROJECT_SOURCE_DIR}/Thirdparty/DBoW2/)
# g2o target
add_subdirectory(${PROJECT_SOURCE_DIR}/Thirdparty/g2o/)

if(USE_CUDA)
  set(OpenCV_DIR "$ENV{OpenCV_DIR}")
  find_package(OpenCV 3.3.0 REQUIRED)
  find_package(Pangolin REQUIRED)

  include_directories(
    ${PROJECT_SOURCE_DIR}
    ${PROJECT_SOURCE_DIR}/include
    ${EIGEN3_INCLUDE_DIR}
    ${Pangolin_INCLUDE_DIRS}
    ${OpenCV_INCLUDE_DIRS}
    ${PROJECT_BINARY_DIR}
  )

  cuda_include_directories(
    ${CUDA_TOOLKIT_ROOT_DIR}/samples/common/inc
  )

  if(USE_VIEWER)
    cuda_add_library(${PROJECT_NAME} SHARED
      src/System.cc
      src/Tracking.cc
      src/LocalMapping.cc
      src/LoopClosing.cc
      src/ORBextractor.cc
      src/ORBmatcher.cc
      src/FrameDrawer.cc
      src/Converter.cc
      src/MapPoint.cc
      src/KeyFrame.cc
      src/Map.cc
      src/MapDrawer.cc
      src/Optimizer.cc
      src/PnPsolver.cc
      src/Frame.cc
      src/KeyFrameDatabase.cc
      src/Sim3Solver.cc
      src/Initializer.cc
      src/Viewer.cc
      src/cuda/Allocator_gpu.cu
      src/cuda/Fast_gpu.cu
      src/cuda/Orb_gpu.cu
      src/cuda/Cuda.cu
    )
    target_link_libraries(${PROJECT_NAME}
      ${OpenCV_LIBS}
      ${EIGEN3_LIBS}
      ${Pangolin_LIBRARIES}
      DBoW2
      g2o
      nvToolsExt
    )
  else(USE_VIEWER)
    cuda_add_library(${PROJECT_NAME} SHARED
      src/System.cc
      src/Tracking.cc
      src/LocalMapping.cc
      src/LoopClosing.cc
      src/ORBextractor.cc
      src/ORBmatcher.cc
      src/Converter.cc
      src/MapPoint.cc
      src/KeyFrame.cc
      src/Map.cc
      src/Optimizer.cc
      src/PnPsolver.cc
      src/Frame.cc
      src/KeyFrameDatabase.cc
      src/Sim3Solver.cc
      src/Initializer.cc
      src/cuda/Allocator_gpu.cu
      src/cuda/Fast_gpu.cu
      src/cuda/Orb_gpu.cu
      src/cuda/Cuda.cu
    )
    target_link_libraries(${PROJECT_NAME}
      ${OpenCV_LIBS}
      ${EIGEN3_LIBS}
      DBoW2
      g2o
      nvToolsExt
    )
  endif(USE_VIEWER)

# Don't use CUDA, build with CPU only
else(USE_CUDA)
  set(OpenCV_DIR "$ENV{OpenCV_DIR}")
  find_package(OpenCV 3.3.0 REQUIRED)
  if(NOT OpenCV_FOUND)
     message(FATAL_ERROR "OpenCV 3.3.0 not found.")
  endif()
  message("OpenCV include dirs are ${OpenCV_INCLUDE_DIRS}")

  if(USE_VIEWER)
    add_library(ORB_SLAM2 SHARED
      src/System.cc
      src/Tracking.cc
      src/LocalMapping.cc
      src/LoopClosing.cc
      src/ORBextractor.cc
      src/ORBmatcher.cc
      src/FrameDrawer.cc
      src/Converter.cc
      src/MapPoint.cc
      src/KeyFrame.cc
      src/Map.cc
      src/MapDrawer.cc
      src/Optimizer.cc
      src/PnPsolver.cc
      src/Frame.cc
      src/KeyFrameDatabase.cc
      src/Sim3Solver.cc
      src/Initializer.cc
      src/Viewer.cc
    )

    target_include_directories(ORB_SLAM2 PUBLIC
      ${PROJECT_SOURCE_DIR}
      ${PROJECT_SOURCE_DIR}/include
      ${OpenCV_INCLUDE_DIRS}
      ${EIGEN3_INCLUDE_DIR}
      ${Pangolin_INCLUDE_DIR}
      ${PROJECT_BINARY_DIR}
    )

    target_link_libraries(ORB_SLAM2
      ${OpenCV_LIBS}
      ${EIGEN3_LIBS}
      ${Pangolin_LIBRARIES}
      DBoW2
      g2o
    )
  else(USE_VIEWER)
    add_library(ORB_SLAM2 SHARED
      src/System.cc
      src/Tracking.cc
      src/LocalMapping.cc
      src/LoopClosing.cc
      src/ORBextractor.cc
      src/ORBmatcher.cc
      src/Converter.cc
      src/MapPoint.cc
      src/KeyFrame.cc
      src/Map.cc
      src/Optimizer.cc
      src/PnPsolver.cc
      src/Frame.cc
      src/KeyFrameDatabase.cc
      src/Sim3Solver.cc
      src/Initializer.cc
    )

    target_include_directories(ORB_SLAM2 PUBLIC
      ${PROJECT_SOURCE_DIR}
      ${PROJECT_SOURCE_DIR}/include
      ${OpenCV_INCLUDE_DIRS}
      ${EIGEN3_INCLUDE_DIR}
      ${PROJECT_BINARY_DIR}
    )

    target_link_libraries(ORB_SLAM2
      ${OpenCV_LIBS}
      ${EIGEN3_LIBS}
      DBoW2
      g2o
    )
  endif(USE_VIEWER)
endif(USE_CUDA)

#Install steps

include(GenerateExportHeader)
set_target_properties(ORB_SLAM2 PROPERTIES CXX_VISIBILITY_PRESET hidden)
set_target_properties(ORB_SLAM2 PROPERTIES VISIBILITY_INLINES_HIDDEN 1)
generate_export_header(ORB_SLAM2)

install(TARGETS ORB_SLAM2
  RUNTIME DESTINATION ${CMAKE_INSTALL_PREFIX}/bin
  ARCHIVE DESTINATION ${CMAKE_INSTALL_PREFIX}/lib
  LIBRARY DESTINATION ${CMAKE_INSTALL_PREFIX}/lib )
install(DIRECTORY ${PROJECT_SOURCE_DIR}/include/
  DESTINATION ${CMAKE_INSTALL_PREFIX}/include/ORB_SLAM2/
  FILES_MATCHING PATTERN "*.h")
install(FILES ${PROJECT_BINARY_DIR}/orb_slam2_export.h 
  DESTINATION ${CMAKE_INSTALL_PREFIX}/include/ORB_SLAM2/)

# Build examples
if(NO_EXAMPLES)
  message(STATUS "Don't build examples")
else(NO_EXAMPLES)
  message(STATUS "Build examples")

  add_executable(rgbd_tum Examples/RGB-D/rgbd_tum.cc)
  target_link_libraries(rgbd_tum ${PROJECT_NAME})

  add_executable(stereo_kitti Examples/Stereo/stereo_kitti.cc)
  target_link_libraries(stereo_kitti ${PROJECT_NAME})

  add_executable(mono_tum Examples/Monocular/mono_tum.cc)
  target_link_libraries(mono_tum ${PROJECT_NAME})

  add_executable(mono_kitti Examples/Monocular/mono_kitti.cc)
  target_link_libraries(mono_kitti ${PROJECT_NAME})

  add_executable(mono_euroc  Examples/Monocular/mono_euroc.cc)
  target_link_libraries(mono_euroc ${PROJECT_NAME})

  if(USE_CUDA)
    # Build the gpu example
    add_executable(tx1 gpu/tx1.cpp)
    target_link_libraries(tx1 ${PROJECT_NAME})
  endif()
endif(NO_EXAMPLES)

install(TARGETS 
    rgbd_tum
    stereo_kitti
    mono_tum
    mono_kitti
  RUNTIME DESTINATION ${CMAKE_INSTALL_PREFIX}/bin )