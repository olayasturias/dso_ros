cmake_minimum_required(VERSION 2.8.3)
project(dso_ros)

# Set the build type.  Options are:
#  Coverage       : w/ debug symbols, w/o optimization, w/ code-coverage
#  Debug          : w/ debug symbols, w/o optimization
#  Release        : w/o debug symbols, w/ optimization
#  RelWithDebInfo : w/ debug symbols, w/ optimization
#  MinSizeRel     : w/o debug symbols, w/ optimization, stripped binaries

find_package(catkin REQUIRED COMPONENTS 
  geometry_msgs
  roscpp
  sensor_msgs
  cv_bridge
)


catkin_package(
  CATKIN_DEPENDS
    geometry_msgs
    roscpp
    sensor_msgs
    cv_bridge
)


include_directories(${catkin_INCLUDE_DIRS})

set(DSO_PATH $ENV{DSO_PATH})
message("\n\n---- LOOKING FOR DSO at\n\"${DSO_PATH}\"")

 

# find_package(Pangolin 0.2 REQUIRED)
# message("\n\n---- FOUND Pangolin Headers at\n\"${Pangolin_INCLUDE_DIRS}\"")
# message("---- FOUND Pangolin Libs at\n\"${Pangolin_LIBRARIES}\"")

find_package(OpenCV 4 REQUIRED)
message("\n\n---- FOUND OpenCV Headers at\n\"${OpenCV_INCLUDE_DIRS}\"")
message("---- FOUND OpenCV Libs at\n\"${OpenCV_LIBS}\"")

find_package(Eigen3 REQUIRED)

find_package(Boost COMPONENTS system thread) 


find_library(DSO_LIBRARY dso ${DSO_PATH}/build/lib)


set(CMAKE_CXX_FLAGS
   "${SSE_FLAGS} -O2 -g -std=c++0x -march=native -fno-omit-frame-pointer"
)

# Sources files
set(dso_SOURCE_FILES
  ${DSO_PATH}/src/FullSystem/FullSystem.cpp
  ${DSO_PATH}/src/FullSystem/FullSystemOptimize.cpp
  ${DSO_PATH}/src/FullSystem/FullSystemOptPoint.cpp
  ${DSO_PATH}/src/FullSystem/FullSystemDebugStuff.cpp
  ${DSO_PATH}/src/FullSystem/FullSystemMarginalize.cpp
  ${DSO_PATH}/src/FullSystem/Residuals.cpp
  ${DSO_PATH}/src/FullSystem/CoarseTracker.cpp
  ${DSO_PATH}/src/FullSystem/CoarseInitializer.cpp
  ${DSO_PATH}/src/FullSystem/ImmaturePoint.cpp
  ${DSO_PATH}/src/FullSystem/HessianBlocks.cpp
  ${DSO_PATH}/src/FullSystem/PixelSelector2.cpp
  ${DSO_PATH}/src/OptimizationBackend/EnergyFunctional.cpp
  ${DSO_PATH}/src/OptimizationBackend/AccumulatedTopHessian.cpp
  ${DSO_PATH}/src/OptimizationBackend/AccumulatedSCHessian.cpp
  ${DSO_PATH}/src/OptimizationBackend/EnergyFunctionalStructs.cpp
  ${DSO_PATH}/src/util/settings.cpp
  ${DSO_PATH}/src/util/Undistort.cpp
  ${DSO_PATH}/src/util/globalCalib.cpp
)

include_directories(
  ${PROJECT_SOURCE_DIR}/src
  ${DSO_PATH}/src
  ${DSO_PATH}/thirdparty/Sophus
  ${OpenCV_INCLUDE_DIRS}
  # ${Pangolin_INCLUDE_DIRS}
  ${EIGEN3_INCLUDE_DIR}
  ${catkin_INCLUDE_DIRS}
)  


set(LIBS
   ${EIGEN3_LIBS}
   ${catkin_LIBRARIES}
  #  ${Pangolin_LIBRARIES}
   ${OpenCV_LIBS}
)

add_executable (dso_live
   src/main.cpp
)

install(TARGETS dso_live
        RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION})


target_link_libraries(dso_live
    ${DSO_LIBRARY}
    # ${Pangolin_LIBRARIES}
    ${OpenCV_LIBRARIES}
    ${catkin_LIBRARIES})

install(DIRECTORY config launch
    DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION})