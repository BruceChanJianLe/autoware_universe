cmake_minimum_required(VERSION 3.14)
project(autoware_elevation_map_loader)

find_package(autoware_cmake REQUIRED)
autoware_package()

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
)
set(Boost_LIBRARY_DIR ${CMAKE_BINARY_DIR}/_install/boost/lib/)
set(Boost_INCLUDE_DIR ${CMAKE_BINARY_DIR}/_install/boost/include/)

find_package(PCL REQUIRED COMPONENTS io)
find_package(Boost REQUIRED)

ament_auto_add_library(${PROJECT_NAME} SHARED
  src/elevation_map_loader_node.cpp
)
target_link_libraries(${PROJECT_NAME} ${PCL_LIBRARIES})

# TODO(wep21): workaround for iron.
# remove this block and update package.xml after iron.
find_package(rosbag2_storage_sqlite3)
target_include_directories(${PROJECT_NAME}
  PRIVATE
    ${rosbag2_storage_sqlite3_INCLUDE_DIRS}
    ${Boost_INCLUDE_DIR}
    # ${CMAKE_BINARY_DIR}/_install/boost/include
)

rclcpp_components_register_node(${PROJECT_NAME}
  PLUGIN "autoware::elevation_map_loader::ElevationMapLoaderNode"
  EXECUTABLE elevation_map_loader_node
)

ament_auto_package(INSTALL_TO_SHARE
  launch
  config
  data
)
