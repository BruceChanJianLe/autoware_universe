cmake_minimum_required(VERSION 3.14)
project(autoware_autonomous_emergency_braking)

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
find_package(PCL REQUIRED)

include_directories(
  include
  SYSTEM
  ${PCL_INCLUDE_DIRS}
)

ament_auto_add_library(autoware_autonomous_emergency_braking_helpers SHARED
  include/autoware/autonomous_emergency_braking/utils.hpp
  src/utils.cpp
)
target_include_directories(autoware_autonomous_emergency_braking_helpers SYSTEM PUBLIC
  ${Boost_INCLUDE_DIR}
)

set(AEB_NODE ${PROJECT_NAME}_node)
ament_auto_add_library(${AEB_NODE} SHARED
  include/autoware/autonomous_emergency_braking/node.hpp
  src/node.cpp
)

target_link_libraries(${AEB_NODE} autoware_autonomous_emergency_braking_helpers)
rclcpp_components_register_node(${AEB_NODE}
  PLUGIN "autoware::motion::control::autonomous_emergency_braking::AEB"
  EXECUTABLE ${PROJECT_NAME}
)

if(BUILD_TESTING)
  ament_add_ros_isolated_gtest(test_aeb
  test/test.cpp)

target_link_libraries(test_aeb ${AEB_NODE})

endif()

ament_auto_package(
  INSTALL_TO_SHARE
  launch
  config
)
