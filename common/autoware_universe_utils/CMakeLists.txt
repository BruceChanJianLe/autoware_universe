cmake_minimum_required(VERSION 3.14)
project(autoware_universe_utils)

option(BUILD_EXAMPLES "Build examples" OFF)

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
find_package(fmt REQUIRED)

ament_auto_add_library(autoware_universe_utils SHARED
  src/geometry/alt_geometry.cpp
  src/geometry/geometry.cpp
  src/geometry/pose_deviation.cpp
  src/geometry/boost_polygon_utils.cpp
  src/geometry/random_convex_polygon.cpp
  src/geometry/random_concave_polygon.cpp
  src/geometry/gjk_2d.cpp
  src/geometry/sat_2d.cpp
  src/math/sin_table.cpp
  src/math/trigonometry.cpp
  src/ros/msg_operation.cpp
  src/ros/marker_helper.cpp
  src/ros/logger_level_configure.cpp
  src/system/backtrace.cpp
  src/system/time_keeper.cpp
  src/geometry/ear_clipping.cpp
)
target_include_directories(autoware_universe_utils PUBLIC ${Boost_INCLUDE_DIR})
target_link_libraries(autoware_universe_utils
  fmt::fmt
)

if(BUILD_TESTING)
  find_package(ament_cmake_ros REQUIRED)

  file(GLOB_RECURSE test_files test/**/*.cpp)

  ament_add_ros_isolated_gtest(test_autoware_universe_utils ${test_files})

  target_link_libraries(test_autoware_universe_utils
    autoware_universe_utils
  )
endif()

if(BUILD_EXAMPLES)
  message(STATUS "Building examples")
  file(GLOB_RECURSE example_files examples/*.cpp)

  foreach(example_file ${example_files})
    get_filename_component(example_name ${example_file} NAME_WE)
    add_executable(${example_name} ${example_file})
    target_link_libraries(${example_name}
      autoware_universe_utils
    )
    install(TARGETS ${example_name}
      DESTINATION lib/${PROJECT_NAME}
    )
  endforeach()
endif()

ament_auto_package()
