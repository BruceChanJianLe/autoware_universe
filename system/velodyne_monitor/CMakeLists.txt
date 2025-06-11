cmake_minimum_required(VERSION 3.14)
project(velodyne_monitor)

include(FetchContent)

FetchContent_Declare(fmt URL https://github.com/fmtlib/fmt/releases/download/8.1.1/fmt-8.1.1.zip)
FetchContent_MakeAvailable(fmt)

find_package(autoware_cmake REQUIRED)
find_package(fmt)
autoware_package()

ament_auto_add_library(${PROJECT_NAME} SHARED
  src/velodyne_monitor.cpp
)

## Specify libraries to link a library or executable target against
target_link_libraries(${PROJECT_NAME} cpprest crypto fmt::fmt)

rclcpp_components_register_node(${PROJECT_NAME}
  PLUGIN "VelodyneMonitor"
  EXECUTABLE ${PROJECT_NAME}_node
)

ament_auto_package(INSTALL_TO_SHARE
  launch
  config
)
