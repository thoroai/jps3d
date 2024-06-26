cmake_minimum_required(VERSION 2.8.12.1) # Version 2.8+ Suppresses warning: "Policy CMP0022 is not set: INTERFACE_LINK_LIBRARIES"
project(jps3d)

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11 -Wall -Wno-deprecated-declarations")

option(BUILD_TESTS "Build the tests." OFF)
# message(STATUS "[jps3d] BUILD_TESTS set to: ${BUILD_TESTS}")

option(INSTALL_PROJECT "Installs the project into standard paths on the system." OFF)
# message(STATUS "[jps3d] INSTALL_PROJECT set to: ${INSTALL_PROJECT}")

if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE RelWithDebInfo)
endif()

#
# Create Target Libraries
#
add_library(jps_lib src/jps_planner/graph_search.cpp
                    src/jps_planner/jps_planner.cpp)
target_link_libraries(jps_lib 
                      PRIVATE Eigen3::Eigen Boost::boost)
target_include_directories(jps_lib
                           PUBLIC $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>)

add_library(dmp_lib src/distance_map_planner/graph_search.cpp
                    src/distance_map_planner/distance_map_planner.cpp)
target_link_libraries(dmp_lib 
                      PRIVATE Eigen3::Eigen Boost::boost)
target_include_directories(dmp_lib
                           PUBLIC $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>)

#
# Build Tests
#
if(BUILD_TESTS)

  add_executable(create_map test/create_map.cpp)
  target_link_libraries(create_map
                        PRIVATE yaml-cpp)

  include(CTest)

  add_executable(test_planner_2d test/test_planner_2d.cpp)
  target_link_libraries(test_planner_2d jps_lib)
  add_test(test_planner_2d test_planner_2d ${CMAKE_SOURCE_DIR}/data/corridor.yaml)

  add_executable(test_planner_3d test/test_planner_3d.cpp)
  target_link_libraries(test_planner_3d jps_lib)
  add_test(test_planner_3d test_planner_3d ${CMAKE_SOURCE_DIR}/data/simple3d.yaml)

  add_executable(test_distance_map_planner_2d test/test_distance_map_planner_2d.cpp)
  target_link_libraries(test_distance_map_planner_2d jps_lib dmp_lib)
  add_test(test_distance_map_planner_2d test_distance_map_planner_2d ${CMAKE_SOURCE_DIR}/data/corridor.yaml)

endif()

#
# Install Project
#
if(INSTALL_PROJECT)

  install(FILES "${PROJECT_NAME}Config.cmake" "${PROJECT_NAME}ConfigVersion.cmake"
    DESTINATION "share/${PROJECT_NAME}/cmake")

  install(TARGETS jps_lib dmp_lib
    RUNTIME DESTINATION bin
    LIBRARY DESTINATION lib
    ARCHIVE DESTINATION lib)

  install(DIRECTORY include/jps_basis include/jps_planner include/jps_collision
          DESTINATION include/)

endif()