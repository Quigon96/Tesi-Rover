cmake_minimum_required(VERSION 3.8)
project(roboclaw_ros2)

set(CMAKE_CXX_STANDARD 14)

## Find catkin macros and libraries
## if COMPONENTS list like find_package(catkin REQUIRED COMPONENTS xyz)
## is used, also find other catkin packages
find_package(ament_cmake REQUIRED)
find_package(rclcpp REQUIRED)
find_package(builtin_interfaces REQUIRED)
find_package(rosidl_default_generators REQUIRED)
find_package(std_msgs REQUIRED)
find_package(geometry_msgs REQUIRED)
find_package(nav_msgs REQUIRED)
find_package(rclpy REQUIRED)
find_package(tf2 REQUIRED)
find_package(tf2_ros REQUIRED)
find_package(tf2_geometry_msgs REQUIRED)
#find_package(roboclaw_ros2 REQUIRED)

#find_package(message_generation REQUIRED)
#find_package(message_runtime REQUIRED)


## System dependencies are found with CMake's conventions
find_package(Boost REQUIRED COMPONENTS system)


## Uncomment this if the package has a setup.py. This macro ensures
## modules and global scripts declared therein get installed
## See http://ros.org/doc/api/catkin/html/user_guide/setup_dot_py.html
# catkin_python_setup()

################################################
## Declare ROS messages, services and actions ##
################################################

## To declare and build messages, services or actions from within this
## package, follow these steps:
## * Let MSG_DEP_SET be the set of packages whose message types you use in
##   your messages/services/actions (e.g. std_msgs, actionlib_msgs, ...).
## * In the file package.xml:
##   * add a build_depend tag for "message_generation"
##   * add a build_depend and a exec_depend tag for each package in MSG_DEP_SET
##   * If MSG_DEP_SET isn't empty the following dependency has been pulled in
##     but can be declared for certainty nonetheless:
##     * add a exec_depend tag for "message_runtime"
## * In this file (CMakeLists.txt):
##   * add "message_generation" and every package in MSG_DEP_SET to
##     find_package(catkin REQUIRED COMPONENTS ...)
##   * add "message_runtime" and every package in MSG_DEP_SET to
##     catkin_package(CATKIN_DEPENDS ...)
##   * uncomment the add_*_files sections below as needed
##     and list every .msg/.srv/.action file to be processed
##   * uncomment the generate_messages entry below
##   * add every package in MSG_DEP_SET to generate_messages(DEPENDENCIES ...)

## Generate messages in the 'msg' folder
#add_message_files(
#        FILES
#        RoboclawMotorVelocity.msg
#        RoboclawEncoderSteps.msg
#)
set(msg_files
        "msg/RoboclawMotorVelocity.msg"
        "msg/RoboclawEncoderSteps.msg"
)

rosidl_generate_interfaces(${PROJECT_NAME}
        ${msg_files}
  DEPENDENCIES geometry_msgs nav_msgs std_msgs 
  builtin_interfaces   # Add packages that above messages depend on, in this case geometry_msgs for Sphere.msg
)
## Generate services in the 'srv' folder
# add_service_files(
#   FILES
#   Service1.srv
#   Service2.srv
# )

## Generate actions in the 'action' folder
# add_action_files(
#   FILES
#   Action1.action
#   Action2.action
# )

## Generate added messages and services with any dependencies listed here


################################################
## Declare ROS dynamic reconfigure parameters ##
################################################

## To declare and build dynamic reconfigure parameters within this
## package, follow these steps:
## * In the file package.xml:
##   * add a build_depend and a exec_depend tag for "dynamic_reconfigure"
## * In this file (CMakeLists.txt):
##   * add "dynamic_reconfigure" to
##     find_package(catkin REQUIRED COMPONENTS ...)
##   * uncomment the "generate_dynamic_reconfigure_options" section below
##     and list every .cfg file to be processed

## Generate dynamic reconfigure parameters in the 'cfg' folder
# generate_dynamic_reconfigure_options(
#   cfg/DynReconf1.cfg
#   cfg/DynReconf2.cfg
# )

###################################
## catkin specific configuration ##
###################################
## The catkin_package macro generates cmake config files for your package
## Declare things to be passed to dependent projects
## INCLUDE_DIRS: uncomment this if your package contains header files
## LIBRARIES: libraries you create in this project that dependent projects also need
## CATKIN_DEPENDS: catkin_packages dependent projects also need
## DEPENDS: system dependencies of this project that dependent projects also need

ament_export_dependencies(geometry_msgs nav_msgs rclcpp rclpy std_msgs tf2 tf2_ros tf2_geometry_msgs)
ament_export_include_directories(include)


###########
## Build ##
###########

## Specify additional locations of header files
## Your package locations should be listed before other locations
include_directories(
        include
   
)

## Declare a C++ library
add_library(roboclaw_driver STATIC
        src/roboclaw_driver.cpp
        src/TimeoutSerial.cpp)
        

add_library(roboclaw_roscore STATIC
        src/roboclaw_roscore.cpp
        )

add_executable(roboclaw_node
        src/roboclaw_node.cpp
        )

add_library(diffdrive_roscore STATIC
        src/diffdrive_roscore.cpp
        )

add_executable(diffdrive_node
        src/diffdrive_node.cpp
        )
        add_library(TimeoutSerial STATIC
        src/TimeoutSerial.cpp
        )     

install(TARGETS
roboclaw_node
diffdrive_node
DESTINATION lib/${PROJECT_NAME}
)
install(FILES include/TimeoutSerial.h include/diffdrive_roscore.h include/roboclaw_driver.h include/roboclaw_roscore.h
DESTINATION lib/${PROJECT_NAME}
)
install(DIRECTORY launch DESTINATION share/${PROJECT_NAME}/)

## Add cmake target dependencies of the library
## as an example, code may need to be generated before libraries
## either from message generation or dynamic reconfigure
# add_dependencies(${PROJECT_NAME} ${${PROJECT_NAME}_EXPORTED_TARGETS} ${catkin_EXPORTED_TARGETS})

## Declare a C++ executable
## With catkin_make all packages are built within a single CMake context
## The recommended prefix ensures that target names across packages don't collide
# add_executable(${PROJECT_NAME}_node src/roboclaw_node.cpp)

## Rename C++ executable without prefix
## The above recommended prefix causes long target names, the following renames the
## target back to the shorter version for ease of user use
## e.g. "rosrun someones_pkg node" instead of "rosrun someones_pkg someones_pkg_node"
# set_target_properties(${PROJECT_NAME}_node PROPERTIES OUTPUT_NAME node PREFIX "")

## Add cmake target dependencies of the executable
## same as for the library above
# add_dependencies(${PROJECT_NAME}_node ${${PROJECT_NAME}_EXPORTED_TARGETS} ${catkin_EXPORTED_TARGETS})

## Specify libraries to link a library or executable target against

ament_target_dependencies(roboclaw_driver
	rclcpp     
        std_msgs
        nav_msgs
        )
        target_link_libraries(roboclaw_driver
        roboclaw_roscore
        )
        rosidl_get_typesupport_target(cpp_typesupport_target ${PROJECT_NAME} "rosidl_typesupport_cpp")
        target_link_libraries(roboclaw_roscore "${cpp_typesupport_target}")   

ament_target_dependencies(roboclaw_roscore
        rclcpp
        std_msgs
        nav_msgs
   #     roboclaw_driver
        )
        target_link_libraries(roboclaw_roscore
        roboclaw_driver
        )
        rosidl_get_typesupport_target(cpp_typesupport_target ${PROJECT_NAME} "rosidl_typesupport_cpp")
        target_link_libraries(roboclaw_roscore "${cpp_typesupport_target}")      


ament_target_dependencies(roboclaw_node
        rclcpp
        std_msgs
        nav_msgs
     #   roboclaw_roscore
        )
        target_link_libraries(roboclaw_node
        roboclaw_roscore
        )
        rosidl_get_typesupport_target(cpp_typesupport_target ${PROJECT_NAME} "rosidl_typesupport_cpp")
        target_link_libraries(roboclaw_node "${cpp_typesupport_target}")   


ament_target_dependencies(diffdrive_roscore
        rclcpp
        std_msgs
        nav_msgs
        tf2
        tf2_ros
        tf2_geometry_msgs
        )
        rosidl_get_typesupport_target(cpp_typesupport_target ${PROJECT_NAME} "rosidl_typesupport_cpp")
        target_link_libraries(diffdrive_roscore "${cpp_typesupport_target}")  


ament_target_dependencies(diffdrive_node
        rclcpp
  std_msgs
  nav_msgs
     #   diffdrive_roscore
        )
        target_link_libraries(diffdrive_node
        diffdrive_roscore
        )
        rosidl_get_typesupport_target(cpp_typesupport_target ${PROJECT_NAME} "rosidl_typesupport_cpp")
        target_link_libraries(diffdrive_node "${cpp_typesupport_target}")  
        
        
        ament_export_dependencies(rosidl_default_runtime)
        ament_export_include_directories(include)
        

ament_package(
     
        # LIBRARIES roboclaw
   
        #  DEPENDS system_lib
)
#############
## Install ##
#############

# all install targets should use catkin DESTINATION variables
# See http://ros.org/doc/api/catkin/html/adv_user_guide/variables.html

## Mark executable scripts (Python etc.) for installation
## in contrast to setup.py, you can choose the destination
# install(PROGRAMS
#   scripts/my_python_script
#   DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
# )

## Mark executables and/or libraries for installation
# install(TARGETS ${PROJECT_NAME} ${PROJECT_NAME}_node
#   ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
#   LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
#   RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
# )

## Mark cpp header files for installation
# install(DIRECTORY include/${PROJECT_NAME}/
#   DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION}
#   FILES_MATCHING PATTERN "*.h"
#   PATTERN ".svn" EXCLUDE
# )

## Mark other files for installation (e.g. launch and bag files, etc.)
# install(FILES
#   # myfile1
#   # myfile2
#   DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}
# )

#############
## Testing ##
#############

## Add gtest based cpp test target and link libraries
# catkin_add_gtest(${PROJECT_NAME}-test test/test_roboclaw.cpp)
# if(TARGET ${PROJECT_NAME}-test)
#   target_link_libraries(${PROJECT_NAME}-test ${PROJECT_NAME})
# endif()

## Add folders to be run by python nosetests
# catkin_add_nosetests(test)
