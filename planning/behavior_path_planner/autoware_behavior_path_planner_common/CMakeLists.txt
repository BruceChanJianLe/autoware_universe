cmake_minimum_required(VERSION 3.14)
project(autoware_behavior_path_planner_common)

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

find_package(autoware_cmake REQUIRED)
autoware_package()

find_package(magic_enum CONFIG REQUIRED)
find_package(Boost REQUIRED)

ament_auto_add_library(${PROJECT_NAME} SHARED
  src/turn_signal_decider.cpp
  src/interface/scene_module_visitor.cpp
  src/interface/scene_module_interface.cpp
  src/interface/scene_module_manager_interface.cpp
  src/utils/utils.cpp
  src/utils/path_utils.cpp
  src/utils/traffic_light_utils.cpp
  src/utils/path_safety_checker/safety_check.cpp
  src/utils/path_safety_checker/objects_filtering.cpp
  src/utils/path_shifter/path_shifter.cpp
  src/utils/drivable_area_expansion/static_drivable_area.cpp
  src/utils/drivable_area_expansion/drivable_area_expansion.cpp
  src/utils/drivable_area_expansion/map_utils.cpp
  src/utils/drivable_area_expansion/footprints.cpp
  src/utils/parking_departure/geometric_parallel_parking.cpp
  src/utils/parking_departure/utils.cpp
  src/utils/occupancy_grid_based_collision_detector/occupancy_grid_based_collision_detector.cpp
  src/marker_utils/utils.cpp
)

target_include_directories(${PROJECT_NAME} SYSTEM PUBLIC
  ${EIGEN3_INCLUDE_DIR}
  ${Boost_INCLUDE_DIR}
)

if(BUILD_TESTING)
  include_directories(${Boost_INCLUDE_DIR})

  ament_add_ros_isolated_gmock(test_${PROJECT_NAME}_utilities
    test/test_utils.cpp
    test/test_path_utils.cpp
    test/test_traffic_light_utils.cpp
  )

  target_link_libraries(test_${PROJECT_NAME}_utilities
  ${PROJECT_NAME}
  )

  ament_add_ros_isolated_gmock(test_${PROJECT_NAME}_drivable_area_expansion
    test/test_drivable_area_expansion.cpp
    test/test_footprints.cpp
    test/test_static_drivable_area.cpp
  )

  target_link_libraries(test_${PROJECT_NAME}_drivable_area_expansion
    ${PROJECT_NAME}
  )

  ament_add_ros_isolated_gmock(test_${PROJECT_NAME}_occupancy_grid_based_collision_detector
    test/test_occupancy_grid_based_collision_detector.cpp
  )

  target_link_libraries(test_${PROJECT_NAME}_occupancy_grid_based_collision_detector
    ${PROJECT_NAME}
  )

  ament_add_ros_isolated_gmock(test_${PROJECT_NAME}_safety_check
    test/test_safety_check.cpp
    test/test_objects_filtering.cpp
  )

  target_link_libraries(test_${PROJECT_NAME}_safety_check
    ${PROJECT_NAME}
  )

  ament_add_ros_isolated_gmock(test_${PROJECT_NAME}_parking_departure
    test/test_parking_departure_utils.cpp
  )

  target_link_libraries(test_${PROJECT_NAME}_parking_departure
    ${PROJECT_NAME}
  )

  ament_add_ros_isolated_gmock(test_${PROJECT_NAME}_path_shifter
    test/test_path_shifter.cpp
  )

  target_link_libraries(test_${PROJECT_NAME}_path_shifter
    ${PROJECT_NAME}
  )

  ament_add_ros_isolated_gmock(test_${PROJECT_NAME}_turn_signal
    test/test_turn_signal.cpp
  )

  target_link_libraries(test_${PROJECT_NAME}_turn_signal
    ${PROJECT_NAME}
  )
endif()

ament_auto_package(INSTALL_TO_SHARE
  test_data
)
