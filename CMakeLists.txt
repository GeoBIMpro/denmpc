cmake_minimum_required(VERSION 2.8.3)

project(denmpc)
add_definitions(-std=c++11)

find_package(catkin REQUIRED COMPONENTS
  roscpp
  rospy
  std_msgs
  message_generation
)

## System dependencies are found with CMake's conventions
find_package(Boost REQUIRED COMPONENTS system)

################################################
## Declare ROS messages, services and actions ##
################################################

## Generate messages in the 'msg' folder
add_message_files(  FILES  VrepInfo.msg)	

## Generate added messages and services with any dependencies listed here
 generate_messages(
   DEPENDENCIES
   std_msgs
 )

###################################
## catkin specific configuration ##
###################################

catkin_package(
  INCLUDE_DIRS ${catkin_INCLUDE_DIRS}
#  LIBRARIES symbolic
  CATKIN_DEPENDS roscpp rospy std_msgs message_runtime
  DEPENDS Boost
)

###########
## Build ##
###########

###################################################################

include_directories(
  src/
  src/user_generated
  src/user_scenarios
  src/controller
  include
)
SET( LIB_CONTROLLER_DIR  src/user_generated)
set(LIB_CONTROLLER_SOURCES 
	src/Agent.cpp 
	src/Constraint.cpp
	src/Coupling.cpp  
	src/controller/Controller.cpp 
	src/controller/Cmscgmres.cpp
	src/MathLib.cpp 
	src/Event.cpp
	src/Scheduler.cpp 
	)
FILE( GLOB CONTROLLERSOURCES ${LIB_CONTROLLER_DIR}/*.cpp )
FOREACH( SRC ${CONTROLLERSOURCES} )
	SET( LIB_CONTROLLER_SOURCES ${LIB_CONTROLLER_SOURCES}  ${SRC} )
ENDFOREACH( SRC ${CONTROLLERSOURCES} )
MESSAGE( STATUS "Generated Controller Sources: " ${CONTROLLERSOURCES} )




SET( LIB_USER_DIR  src/user_scenarios)
FILE( GLOB USER_CODE ${LIB_USER_DIR}/*.cpp )
MESSAGE( STATUS "All USER Sources: " ${USER_CODE} )

FOREACH( SRC ${USER_CODE} )
SET(NODENAME ${SRC})
get_filename_component(NODENAME ${SRC} NAME_WE)
SET(NODENAME scenario_${NODENAME}_node)
add_executable(${NODENAME} ${SRC}  ${LIB_CONTROLLER_SOURCES})
add_dependencies(${NODENAME} ${${PROJECT_NAME}_EXPORTED_TARGETS} ${catkin_EXPORTED_TARGETS})
target_link_libraries(${NODENAME} ${catkin_LIBRARIES} )
ENDFOREACH( SRC ${USER_CODE} )
