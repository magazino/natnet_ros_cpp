cmake_minimum_required(VERSION 3.0.2)
project(natnet_ros_cpp)

#################
## catkin deps ##
#################
find_package(catkin REQUIRED COMPONENTS
  geometry_msgs
  message_generation
  roscpp
  rospy
  sensor_msgs
  std_msgs
  tf2 
  tf2_ros
  visualization_msgs
)

####################
## 3rd party deps ##
####################
find_package(Boost 1.65 REQUIRED COMPONENTS system thread)
find_package(Eigen3 REQUIRED)

# Download Natnet SDK.
execute_process(
	COMMAND chmod "+x" "${CMAKE_CURRENT_SOURCE_DIR}/install_sdk.sh"
	WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
)
execute_process(
	COMMAND bash "-c" "${CMAKE_CURRENT_SOURCE_DIR}/install_sdk.sh"
	WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
)
set(NATNET_SDK_DIR ${CMAKE_CURRENT_SOURCE_DIR}/deps/NatNetSDK/) 
set(NATNET_SDK_LIBRARY ${CMAKE_CURRENT_SOURCE_DIR}/deps/NatNetSDK/lib/libNatNet.so)

set(THIRD_PARTY_INCLUDE_DIRS
  include/natnet_ros_cpp
  ${Boost_INCLUDE_DIRS}
  ${EIGEN3_INCLUDE_DIRS}
  ${NATNET_SDK_DIR}/include
)

include_directories(
  ${catkin_INCLUDE_DIRS}
  ${THIRD_PARTY_INCLUDE_DIRS}
)

###############
## msg / srv ##
###############
add_service_files(
  FILES
  MarkerPoses.srv
)
generate_messages(
  DEPENDENCIES
  std_msgs
)

catkin_package(
  CATKIN_DEPENDS
  geometry_msgs
  message_runtime
  roscpp
  rospy
  sensor_msgs
  std_msgs
)

##################
## Exectutables ##
##################
add_executable(${PROJECT_NAME} src/natnet_ros.cpp src/internal.cpp src/nn_filter.cpp)
target_link_libraries(${PROJECT_NAME} ${catkin_LIBRARIES} ${NATNET_SDK_LIBRARY})

add_executable(marker_poses_server src/marker_poses_server.cpp)
target_link_libraries(marker_poses_server ${catkin_LIBRARIES} ${NATNET_SDK_LIBRARY})
add_dependencies(marker_poses_server ${PROJECT_NAME})

#############
## Install ##
#############

install(DIRECTORY
  config
  launch
  ui
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}
)

catkin_install_python(PROGRAMS
  scripts/helper_node.py
  DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
)

install(TARGETS
  ${PROJECT_NAME}
  marker_poses_server
  RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
)

# Install shared library file of closed source NatNet SDK
install(FILES ${NATNET_SDK_LIBRARY} DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION})
