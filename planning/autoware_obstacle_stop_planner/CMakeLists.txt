cmake_minimum_required(VERSION 3.14)
project(autoware_obstacle_stop_planner)

find_package(autoware_cmake REQUIRED)
autoware_package()

message( "External project - Boost" )

include(ExternalProject)

ExternalProject_Add(
    Boost
    URL "https://archives.boost.io/release/1.74.0/source/boost_1_74_0.tar.bz2"
    URL_HASH SHA256=83bfc1507731a0906e387fc28b7ef5417d591429e51e788417fe9ff025e116b1
    PREFIX ${CMAKE_BINARY_DIR}/_deps/boost
    CONFIGURE_COMMAND ./bootstrap.sh --prefix=${CMAKE_BINARY_DIR}/_install/boost
    BUILD_COMMAND ./b2 install
    BUILD_IN_SOURCE 1
    INSTALL_DIR ${CMAKE_BINARY_DIR}/_install/boost
    INSTALL_COMMAND ""
    LOG_CONFIGURE ON
    LOG_BUILD ON
    LOG_INSTALL ON
    DOWNLOAD_EXTRACT_TIMESTAMP TRUE
)
set(Boost_LIBRARY_DIR ${CMAKE_BINARY_DIR}/_install/boost/lib/)
set(Boost_INCLUDE_DIR ${CMAKE_BINARY_DIR}/_install/boost/include/)

find_package(Boost REQUIRED)
find_package(Eigen3 REQUIRED)
find_package(PCL REQUIRED)

ament_auto_add_library(${PROJECT_NAME} SHARED
  DIRECTORY
  src
)

target_include_directories(${PROJECT_NAME}
  SYSTEM PUBLIC
    ${PCL_INCLUDE_DIRS}
    ${EIGEN3_INCLUDE_DIR}
    ${Boost_INCLUDE_DIR}
  INTERFACE
    src
)

target_link_libraries(${PROJECT_NAME}
    ${PCL_LIBRARIES}
)

rclcpp_components_register_node(${PROJECT_NAME}
  PLUGIN "autoware::motion_planning::ObstacleStopPlannerNode"
  EXECUTABLE ${PROJECT_NAME}_node
)

if(BUILD_TESTING)
  ament_add_ros_isolated_gtest(test_${PROJECT_NAME}
    test/test_${PROJECT_NAME}_node_interface.cpp
  )
  target_link_libraries(test_${PROJECT_NAME}
    ${PROJECT_NAME}
  )
endif()

ament_auto_package(
  INSTALL_TO_SHARE
  config
  launch
)
