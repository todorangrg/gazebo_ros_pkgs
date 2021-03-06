cmake_minimum_required(VERSION 2.8.3)
project(gazebo_plugins)

find_package(catkin REQUIRED COMPONENTS 
  message_generation 
  gazebo_msgs 
  roscpp 
  rospy 
  nodelet 
  angles 
  std_srvs 
  geometry_msgs 
  sensor_msgs 
  nav_msgs 
  urdf 
  tf 
  tf2_ros 
  dynamic_reconfigure 
  driver_base 
  rosgraph_msgs 
  trajectory_msgs 
  image_transport 
  rosconsole
  cv_bridge
  polled_camera
  diagnostic_updater
  camera_info_manager
)

include (FindPkgConfig)
if (PKG_CONFIG_FOUND)
  pkg_check_modules(XML libxml-2.0)
  pkg_check_modules(OGRE OGRE)
  pkg_check_modules(OGRE-Terrain OGRE-Terrain)
else()
  message(FATAL_ERROR "pkg-config is required; please install it")
endif()

# Depend on system install of Gazebo and SDFormat
find_package(gazebo REQUIRED)
find_package(Boost REQUIRED COMPONENTS thread)

execute_process(COMMAND
  pkg-config --variable=plugindir OGRE
  OUTPUT_VARIABLE OGRE_PLUGIN_PATH
  OUTPUT_STRIP_TRAILING_WHITESPACE
)

catkin_python_setup()

generate_dynamic_reconfigure_options(
  cfg/CameraSynchronizer.cfg
  cfg/GazeboRosCamera.cfg
  cfg/GazeboRosOpenniKinect.cfg
  cfg/Hokuyo.cfg
)

include_directories(include
  ${Boost_INCLUDE_DIRS}
  ${catkin_INCLUDE_DIRS}
  ${GAZEBO_INCLUDE_DIRS}
  ${OGRE_INCLUDE_DIRS}
  ${OGRE-Terrain_INCLUDE_DIRS}
)

link_directories(
  ${GAZEBO_LIBRARY_DIRS}
)

catkin_package(
  INCLUDE_DIRS include
  LIBRARIES 
  vision_reconfigure 
  gazebo_ros_utils 
  gazebo_ros_camera_utils 
  gazebo_ros_camera 
  gazebo_ros_multicamera 
  gazebo_ros_depth_camera 
  gazebo_ros_openni_kinect 
  gazebo_ros_gpu_laser 
  gazebo_ros_laser 
  gazebo_ros_block_laser 
  gazebo_ros_p3d 
  gazebo_ros_imu 
  gazebo_ros_f3d 
  gazebo_ros_ft_sensor
  gazebo_ros_bumper 
  gazebo_ros_template 
  gazebo_ros_projector 
  gazebo_ros_prosilica 
  gazebo_ros_force 
  gazebo_ros_joint_trajectory 
  gazebo_ros_joint_state_publisher 
  gazebo_ros_joint_pose_trajectory
  gazebo_ros_diff_drive
  gazebo_ros_tricycle_drive
  gazebo_ros_skid_steer_drive
  gazebo_ros_video
  gazebo_ros_planar_move
  
  CATKIN_DEPENDS 
  message_generation 
  gazebo_msgs 
  roscpp 
  rospy 
  nodelet 
  angles 
  std_srvs 
  geometry_msgs 
  sensor_msgs 
  nav_msgs 
  urdf 
  tf 
  tf2_ros
  dynamic_reconfigure 
  driver_base 
  rosgraph_msgs 
  trajectory_msgs 
  image_transport 
  rosconsole
  camera_info_manager
  DEPENDS 
    gazebo 
    SDF
  )
add_dependencies(${PROJECT_NAME}_gencfg ${catkin_EXPORTED_TARGETS})

## Executables
add_executable(hokuyo_node src/hokuyo_node.cpp)
add_dependencies(hokuyo_node ${PROJECT_NAME}_gencfg)
target_link_libraries(hokuyo_node
  ${catkin_LIBRARIES}
)

add_library(gazebo_ros_utils src/gazebo_ros_utils.cpp)
target_link_libraries(gazebo_ros_utils ${GAZEBO_LIBRARIES} ${catkin_LIBRARIES} ${Boost_LIBRARIES})

add_library(vision_reconfigure src/vision_reconfigure.cpp)
add_dependencies(vision_reconfigure ${PROJECT_NAME}_gencfg)
target_link_libraries(vision_reconfigure ${catkin_LIBRARIES})

add_executable(camera_synchronizer src/camera_synchronizer.cpp)
add_dependencies(camera_synchronizer ${PROJECT_NAME}_gencfg)
target_link_libraries(camera_synchronizer vision_reconfigure ${catkin_LIBRARIES})

add_executable(pub_joint_trajectory_test test/pub_joint_trajectory_test.cpp)
target_link_libraries(pub_joint_trajectory_test ${GAZEBO_LIBRARIES} ${catkin_LIBRARIES} ${Boost_LIBRARIES})
add_dependencies(pub_joint_trajectory_test ${catkin_EXPORTED_TARGETS}) # don't build until gazebo_msgs is done

add_definitions(-fPIC) # what is this for?

## Plugins
add_library(gazebo_ros_camera_utils src/gazebo_ros_camera_utils.cpp)
add_dependencies(gazebo_ros_camera_utils ${PROJECT_NAME}_gencfg)
target_link_libraries(gazebo_ros_camera_utils ${GAZEBO_LIBRARIES} ${catkin_LIBRARIES} ${Boost_LIBRARIES})

add_library(MultiCameraPlugin src/MultiCameraPlugin.cpp)
target_link_libraries(MultiCameraPlugin ${GAZEBO_LIBRARIES} ${catkin_LIBRARIES} ${Boost_LIBRARIES})

add_library(gazebo_ros_camera src/gazebo_ros_camera.cpp)
add_dependencies(gazebo_ros_camera ${PROJECT_NAME}_gencfg)
target_link_libraries(gazebo_ros_camera gazebo_ros_camera_utils ${GAZEBO_LIBRARIES} CameraPlugin ${catkin_LIBRARIES})

add_library(gazebo_ros_multicamera src/gazebo_ros_multicamera.cpp)
add_dependencies(gazebo_ros_multicamera ${PROJECT_NAME}_gencfg)
target_link_libraries(gazebo_ros_multicamera gazebo_ros_camera_utils ${GAZEBO_LIBRARIES} MultiCameraPlugin ${catkin_LIBRARIES})

add_library(gazebo_ros_depth_camera src/gazebo_ros_depth_camera.cpp)
add_dependencies(gazebo_ros_depth_camera ${PROJECT_NAME}_gencfg)
target_link_libraries(gazebo_ros_depth_camera gazebo_ros_camera_utils ${GAZEBO_LIBRARIES} DepthCameraPlugin ${catkin_LIBRARIES})

add_library(gazebo_ros_openni_kinect src/gazebo_ros_openni_kinect.cpp)
add_dependencies(gazebo_ros_openni_kinect ${PROJECT_NAME}_gencfg)
target_link_libraries(gazebo_ros_openni_kinect gazebo_ros_camera_utils ${GAZEBO_LIBRARIES} DepthCameraPlugin ${catkin_LIBRARIES})

add_library(gazebo_ros_gpu_laser src/gazebo_ros_gpu_laser.cpp)
target_link_libraries(gazebo_ros_gpu_laser ${catkin_LIBRARIES} ${GAZEBO_LIBRARIES} GpuRayPlugin)

add_library(gazebo_ros_laser src/gazebo_ros_laser.cpp)
target_link_libraries(gazebo_ros_laser ${GAZEBO_LIBRARIES} RayPlugin ${catkin_LIBRARIES})

add_library(gazebo_ros_block_laser src/gazebo_ros_block_laser.cpp)
target_link_libraries(gazebo_ros_block_laser ${GAZEBO_LIBRARIES} RayPlugin ${catkin_LIBRARIES})

add_library(gazebo_ros_p3d src/gazebo_ros_p3d.cpp)
target_link_libraries(gazebo_ros_p3d ${GAZEBO_LIBRARIES} ${catkin_LIBRARIES} ${Boost_LIBRARIES})

add_library(gazebo_ros_imu src/gazebo_ros_imu.cpp)
target_link_libraries(gazebo_ros_imu ${GAZEBO_LIBRARIES} ${catkin_LIBRARIES} ${Boost_LIBRARIES})

add_library(gazebo_ros_f3d src/gazebo_ros_f3d.cpp)
target_link_libraries(gazebo_ros_f3d ${GAZEBO_LIBRARIES} ${catkin_LIBRARIES} ${Boost_LIBRARIES})

add_library(gazebo_ros_bumper src/gazebo_ros_bumper.cpp)
add_dependencies(gazebo_ros_bumper gazebo_msgs_gencpp)
target_link_libraries(gazebo_ros_bumper ${GAZEBO_LIBRARIES} ${Boost_LIBRARIES} ContactPlugin ${catkin_LIBRARIES})

add_library(gazebo_ros_projector src/gazebo_ros_projector.cpp)
target_link_libraries(gazebo_ros_projector ${GAZEBO_LIBRARIES} ${Boost_LIBRARIES} ${catkin_LIBRARIES})

add_library(gazebo_ros_prosilica src/gazebo_ros_prosilica.cpp)
add_dependencies(gazebo_ros_prosilica ${PROJECT_NAME}_gencfg)
target_link_libraries(gazebo_ros_prosilica gazebo_ros_camera_utils ${GAZEBO_LIBRARIES} CameraPlugin ${catkin_LIBRARIES})

add_library(gazebo_ros_force src/gazebo_ros_force.cpp)
target_link_libraries(gazebo_ros_force ${GAZEBO_LIBRARIES} ${catkin_LIBRARIES} ${Boost_LIBRARIES})

add_library(gazebo_ros_joint_trajectory src/gazebo_ros_joint_trajectory.cpp)
add_dependencies(gazebo_ros_joint_trajectory gazebo_msgs_gencpp)
target_link_libraries(gazebo_ros_joint_trajectory ${GAZEBO_LIBRARIES} ${catkin_LIBRARIES} ${Boost_LIBRARIES})


add_library(gazebo_ros_joint_state_publisher src/gazebo_ros_joint_state_publisher.cpp)
set_target_properties(gazebo_ros_joint_state_publisher PROPERTIES LINK_FLAGS "${ld_flags}")
set_target_properties(gazebo_ros_joint_state_publisher PROPERTIES COMPILE_FLAGS "${cxx_flags}")
add_dependencies(gazebo_ros_joint_state_publisher gazebo_msgs_gencpp)
target_link_libraries(gazebo_ros_joint_state_publisher ${GAZEBO_LIBRARIES} ${SDFormat_LIBRARIES} ${catkin_LIBRARIES} ${Boost_LIBRARIES})

add_library(gazebo_ros_joint_pose_trajectory src/gazebo_ros_joint_pose_trajectory.cpp)
add_dependencies(gazebo_ros_joint_pose_trajectory gazebo_msgs_gencpp)
target_link_libraries(gazebo_ros_joint_pose_trajectory ${GAZEBO_LIBRARIES} ${catkin_LIBRARIES} ${Boost_LIBRARIES})

add_library(gazebo_ros_diff_drive src/gazebo_ros_diff_drive.cpp)
target_link_libraries(gazebo_ros_diff_drive gazebo_ros_utils ${GAZEBO_LIBRARIES} ${catkin_LIBRARIES} ${Boost_LIBRARIES})

add_library(gazebo_ros_tricycle_drive src/gazebo_ros_tricycle_drive.cpp)
target_link_libraries(gazebo_ros_tricycle_drive gazebo_ros_utils ${GAZEBO_LIBRARIES} ${Boost_LIBRARIES} ${catkin_LIBRARIES})

add_library(gazebo_ros_skid_steer_drive src/gazebo_ros_skid_steer_drive.cpp)
target_link_libraries(gazebo_ros_skid_steer_drive ${GAZEBO_LIBRARIES} ${catkin_LIBRARIES} ${Boost_LIBRARIES})

add_library(gazebo_ros_video src/gazebo_ros_video.cpp)
target_link_libraries(gazebo_ros_video ${GAZEBO_LIBRARIES} ${catkin_LIBRARIES} ${Boost_LIBRARIES} ${OGRE_LIBRARIES})

add_library(gazebo_ros_planar_move src/gazebo_ros_planar_move.cpp)
target_link_libraries(gazebo_ros_planar_move ${GAZEBO_LIBRARIES} ${catkin_LIBRARIES} ${Boost_LIBRARIES})

add_library(gazebo_ros_hand_of_god src/gazebo_ros_hand_of_god.cpp)
set_target_properties(gazebo_ros_hand_of_god PROPERTIES LINK_FLAGS "${ld_flags}")
set_target_properties(gazebo_ros_hand_of_god PROPERTIES COMPILE_FLAGS "${cxx_flags}")
target_link_libraries(gazebo_ros_hand_of_god ${GAZEBO_LIBRARIES} ${SDFormat_LIBRARIES} ${catkin_LIBRARIES} ${Boost_LIBRARIES})

add_library(gazebo_ros_ft_sensor src/gazebo_ros_ft_sensor.cpp)
target_link_libraries(gazebo_ros_ft_sensor ${GAZEBO_LIBRARIES} ${catkin_LIBRARIES} ${Boost_LIBRARIES})

##
## Add your new plugin here
##

## Template
add_library(gazebo_ros_template src/gazebo_ros_template.cpp)
target_link_libraries(gazebo_ros_template ${GAZEBO_LIBRARIES} ${catkin_LIBRARIES} ${Boost_LIBRARIES})

install(TARGETS 
  hokuyo_node 
  vision_reconfigure 
  camera_synchronizer
  gazebo_ros_utils
  gazebo_ros_camera_utils
  gazebo_ros_camera
  gazebo_ros_multicamera
  MultiCameraPlugin
  gazebo_ros_depth_camera
  gazebo_ros_openni_kinect
  gazebo_ros_openni_kinect
  gazebo_ros_laser
  gazebo_ros_block_laser
  gazebo_ros_p3d
  gazebo_ros_imu
  gazebo_ros_f3d
  gazebo_ros_ft_sensor
  gazebo_ros_bumper
  gazebo_ros_hand_of_god
  gazebo_ros_template
  gazebo_ros_projector
  gazebo_ros_prosilica
  gazebo_ros_force
  gazebo_ros_joint_trajectory
  gazebo_ros_joint_state_publisher
  gazebo_ros_joint_pose_trajectory
  gazebo_ros_diff_drive
  gazebo_ros_tricycle_drive
  gazebo_ros_skid_steer_drive
  gazebo_ros_video
  gazebo_ros_planar_move
  pub_joint_trajectory_test  
  gazebo_ros_gpu_laser
  DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
  LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  )

install(DIRECTORY include/
   DESTINATION ${CATKIN_GLOBAL_INCLUDE_DESTINATION}
   FILES_MATCHING PATTERN "*.h"
)

install(PROGRAMS scripts/set_wrench.py scripts/set_pose.py scripts/gazebo_model
  DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
  )

install(DIRECTORY Media
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}
  )

install(DIRECTORY test
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}
  )
