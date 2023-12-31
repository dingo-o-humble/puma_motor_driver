################################################################################
# Set minimum required version of cmake, project name and compile options
################################################################################
cmake_minimum_required(VERSION 3.5)
project(puma_motor_driver)

if (NOT CMAKE_CXX_STANDARD)
    set(CMAKE_CXX_STANDARD 14)
endif ()
if (CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
    add_compile_options(-Wall -Wextra -Wpedantic -fPIC)
endif ()

################################################################################
# Find ament packages and libraries for ament and system dependencies
################################################################################
find_package(ament_cmake REQUIRED)
find_package(diagnostic_updater REQUIRED)
find_package(puma_motor_msgs REQUIRED)
find_package(rclcpp REQUIRED)
find_package(std_msgs REQUIRED)
find_package(sensor_msgs REQUIRED)
find_package(serial_driver REQUIRED)
find_package(socketcan_interface REQUIRED)

################################################################################
# Build
################################################################################

################## Boost Settings ##################
set(Boost_USE_STATIC_LIBS OFF)
set(Boost_USE_MULTITHREADED ON)
set(Boost_USE_STATIC_RUNTIME OFF)
find_package(Boost REQUIRED)
####################################################

include_directories(
  include
	${Boost_INCLUDE_DIRS}
)

ament_export_include_directories(include)

add_library(puma_motor_driver SHARED
  src/driver.cpp
  #src/serial_gateway.cpp
  src/socketcan_gateway.cpp
  src/multi_driver_node.cpp
  src/diagnostic_updater.cpp
)

target_link_libraries(puma_motor_driver ${Boost_LIBRARIES})

set(DEPENDENCIES
	ament_cmake
	diagnostic_updater
	puma_motor_msgs
	rclcpp
	std_msgs
	sensor_msgs
	serial_driver
  socketcan_interface
)


ament_target_dependencies(puma_motor_driver ${DEPENDENCIES})
ament_export_libraries(puma_motor_driver)

add_executable(puma_multi_controller_node src/multi_controller_node.cpp)
ament_target_dependencies(puma_multi_controller_node ${DEPENDENCIES})
target_link_libraries(puma_multi_controller_node puma_motor_driver)

################################################################################
# Install
################################################################################
install(TARGETS puma_motor_driver puma_multi_controller_node
	ARCHIVE DESTINATION lib
	LIBRARY DESTINATION lib
	RUNTIME DESTINATION bin
)

install(TARGETS puma_motor_driver
  EXPORT export_${PROJECT_NAME}
  DESTINATION lib)

install(
  DIRECTORY launch
  DESTINATION share/${PROJECT_NAME}
)

install(DIRECTORY include/${PROJECT_NAME}/
  DESTINATION include/${PROJECT_NAME})

################################################################################
# Macro for ament package
################################################################################
ament_export_dependencies(${DEPENDENCIES})
ament_package()

