cmake_minimum_required(VERSION 3.5.1)
project(grid_map_core)

set(CMAKE_CXX_FLAGS "-std=c++11 ${CMAKE_CXX_FLAGS}")
add_compile_options(-Wall -Wextra -Wpedantic)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

## Define Eigen addons. 
include(cmake/${PROJECT_NAME}-extras.cmake)

## System dependencies are found with CMake's conventions
#find_package(Eigen3 REQUIRED)
# Solution to find Eigen3 with Saucy.
find_package(Eigen3 QUIET)
if(NOT EIGEN3_FOUND)
  find_package(PkgConfig REQUIRED)
  pkg_check_modules(EIGEN3 REQUIRED eigen3)
  set(EIGEN3_INCLUDE_DIR ${EIGEN3_INCLUDE_DIRS})
endif()

###########
## Build ##
###########

## Specify additional locations of header files
include_directories(
  include
  SYSTEM
    ${catkin_INCLUDE_DIRS}
    ${EIGEN3_INCLUDE_DIR}
)

## Declare a cpp library
add_library(${PROJECT_NAME} SHARED
   src/GridMap.cpp
   src/GridMapMath.cpp
   src/SubmapGeometry.cpp
   src/BufferRegion.cpp
   src/Polygon.cpp
   src/CubicInterpolation.cpp
   src/iterators/GridMapIterator.cpp
   src/iterators/SubmapIterator.cpp
   src/iterators/CircleIterator.cpp
   src/iterators/EllipseIterator.cpp
   src/iterators/SpiralIterator.cpp
   src/iterators/PolygonIterator.cpp
   src/iterators/LineIterator.cpp
   src/iterators/SlidingWindowIterator.cpp
)

target_link_libraries(${PROJECT_NAME}
  ${catkin_LIBRARIES}
)

#############
## Install ##
#############

# Mark executables and/or libraries for installation
install(
  TARGETS ${PROJECT_NAME}
  ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
)

#############
## Testing ##
#############

if(CATKIN_ENABLE_TESTING)
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -pthread")

  ## Add gtest based cpp test target and link libraries
  catkin_add_gtest(${PROJECT_NAME}-test
    test/test_grid_map_core.cpp
    test/test_helpers.cpp
    test/CubicConvolutionInterpolationTest.cpp
    test/CubicInterpolationTest.cpp
    test/GridMapMathTest.cpp
    test/GridMapTest.cpp
    test/GridMapIteratorTest.cpp
    test/LineIteratorTest.cpp
    test/EllipseIteratorTest.cpp
    test/SubmapIteratorTest.cpp
    test/PolygonIteratorTest.cpp
    test/PolygonTest.cpp
    test/EigenPluginsTest.cpp
    test/SpiralIteratorTest.cpp
    test/SlidingWindowIteratorTest.cpp
  )
  target_include_directories(${PROJECT_NAME}-test PRIVATE
    include
  )
  target_include_directories(${PROJECT_NAME}-test SYSTEM PUBLIC
    ${catkin_INCLUDE_DIRS}
    ${EIGEN3_INCLUDE_DIR}
  )
  target_link_libraries(${PROJECT_NAME}-test
    ${PROJECT_NAME}
  )

  ###################
  ## Code_coverage ##
  ###################
  find_package(cmake_code_coverage QUIET)
  if(cmake_code_coverage_FOUND)
    add_gtest_coverage(
      TEST_BUILD_TARGETS
        ${PROJECT_NAME}-test
    )
  endif()
endif()

#################
## Clang_tools ##
#################
find_package(cmake_clang_tools QUIET)
if(cmake_clang_tools_FOUND)
  add_default_clang_tooling(
    DISABLE_CLANG_FORMAT
  )
endif(cmake_clang_tools_FOUND)
