cmake_minimum_required(VERSION 2.8)
project(jps3d)

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11 -Wall -Wno-deprecated-declarations")

IF(NOT CMAKE_BUILD_TYPE)
  SET(CMAKE_BUILD_TYPE RelWithDebInfo)
ENDIF()

set(BUILD_SHARED_LIBS ON)

FIND_PACKAGE(Boost REQUIRED)
# FIND_PACKAGE(BOOST REQUIRED)
FIND_PACKAGE(Eigen3 REQUIRED)
# FIND_PACKAGE(EIGEN3 REQUIRED)
FIND_PACKAGE(PkgConfig REQUIRED)
PKG_CHECK_MODULES(YAMLCPP REQUIRED yaml-cpp)

# IF(Boost_FOUND)
IF(BOOST_FOUND)
  MESSAGE(STATUS ">> Found Boost.")
ELSE()
  # MESSAGE(FATAL_ERROR ">> Could not find Boost.")
  MESSAGE(">> Could not find Boost.")
ENDIF()
message("Boost OUTPUT: " ${Boost_LIBRARIES})
message("Boost OUTPUT: " ${Boost_INCLUDE_DIRS})

# IF(Eigen3_FOUND)
IF(EIGEN3_FOUND)
  MESSAGE(STATUS ">> Found EIGEN3.")
ELSE()
  # MESSAGE(FATAL_ERROR ">> Could not find EIGEN3.")
  MESSAGE(">> Could not find EIGEN3.")
ENDIF()
# set(EIGEN3_INCLUDE_DIRS "/home/rvishwanatha/work/thoro/knight/.toolchain/x86_64-linux/2840a1151635c76bcd4b0b532a1f7347af320c9e9aee39adce336e27c1dd1f96/packages/eigen/d85d2c4588960ea46507925bd7c9273fdf582c602d25dbf1ce9be772d6680914/install/include/eigen3")
set(EIGEN3_INCLUDE_DIRS "/usr/include/eigen3")
message("Eigen OUTPUT: " ${EIGEN3_INCLUDE_DIRS})

IF(YAMLCPP_FOUND)
  MESSAGE(STATUS "Found yaml-cpp.")
ELSE()
  MESSAGE(FATAL_ERROR "Could not find yaml-cpp.")
ENDIF()
set(YAMLCPP_INCLUDE_DIRS "/usr/include/")
message("Yaml-cpp OUTPUT: " ${YAMLCPP_INCLUDE_DIRS})

# INCLUDE_DIRECTORIES(include ${EIGEN3_INCLUDE_DIRS} ${Boost_INCLUDE_DIRS})
# INCLUDE_DIRECTORIES(include ${EIGEN3_INCLUDE_DIRS} ${Boost_INCLUDE_DIRS})
INCLUDE_DIRECTORIES(include ${EIGEN3_INCLUDE_DIRS} ${Boost_INCLUDE_DIRS} ${YAMLCPP_INCLUDE_DIRS})

add_library(jps_lib src/jps_planner/graph_search.cpp
  src/jps_planner/jps_planner.cpp)

add_library(dmp_lib src/distance_map_planner/graph_search.cpp
  src/distance_map_planner/distance_map_planner.cpp)

add_executable(create_map test/create_map.cpp)
target_link_libraries(create_map ${YAMLCPP_LIBRARIES})

include(CTest)

add_executable(test_planner_2d test/test_planner_2d.cpp)
target_link_libraries(test_planner_2d jps_lib ${YAMLCPP_LIBRARIES} ${Boost_LIBRARIES})
# target_link_libraries(test_planner_2d jps_lib ${YAMLCPP_LIBRARIES} Boost::boost)
add_test(test_planner_2d test_planner_2d ${CMAKE_SOURCE_DIR}/data/corridor.yaml)

add_executable(test_planner_3d test/test_planner_3d.cpp)
target_link_libraries(test_planner_3d jps_lib ${YAMLCPP_LIBRARIES})
add_test(test_planner_3d test_planner_3d ${CMAKE_SOURCE_DIR}/data/simple3d.yaml)

add_executable(test_distance_map_planner_2d test/test_distance_map_planner_2d.cpp)
target_link_libraries(test_distance_map_planner_2d jps_lib dmp_lib ${YAMLCPP_LIBRARIES} ${Boost_LIBRARIES})
# target_link_libraries(test_distance_map_planner_2d jps_lib dmp_lib ${YAMLCPP_LIBRARIES} Boost::boost)
add_test(test_distance_map_planner_2d test_distance_map_planner_2d ${CMAKE_SOURCE_DIR}/data/corridor.yaml)


install(FILES "${PROJECT_NAME}Config.cmake" "${PROJECT_NAME}ConfigVersion.cmake"
  DESTINATION "share/${PROJECT_NAME}/cmake")

install(TARGETS jps_lib dmp_lib
  RUNTIME DESTINATION bin
  LIBRARY DESTINATION lib
  ARCHIVE DESTINATION lib)

install(DIRECTORY include/jps_basis include/jps_planner include/jps_collision
  DESTINATION include)
