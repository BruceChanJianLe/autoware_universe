cmake_minimum_required(VERSION 3.14)
project(predicted_path_checker)

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

include_directories(
  include
  SYSTEM
  ${Eigen3_INCLUDE_DIRS}
)

ament_auto_add_library(predicted_path_checker SHARED
  src/predicted_path_checker_node/predicted_path_checker_node.cpp
  src/predicted_path_checker_node/collision_checker.cpp
  src/predicted_path_checker_node/utils.cpp
  src/predicted_path_checker_node/debug_marker.cpp

)
target_include_directories(predicted_path_checker SYSTEM PUBLIC
  ${Boost_INCLUDE_DIR}
)

rclcpp_components_register_node(predicted_path_checker
  PLUGIN "autoware::motion::control::predicted_path_checker::PredictedPathCheckerNode"
  EXECUTABLE predicted_path_checker_node
)

if(BUILD_TESTING)
  find_package(ament_lint_auto REQUIRED)
  ament_lint_auto_find_test_dependencies()
endif()

ament_auto_package(
  INSTALL_TO_SHARE
  launch
  config
)
