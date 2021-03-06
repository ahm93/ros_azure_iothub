# Copyright (c) Microsoft Corporation. All rights reserved.
# Licensed under the MIT License.

cmake_minimum_required(VERSION 2.8.3)
project(roscpp_azure_iothub)

if(MSVC)
  add_definitions(-D_CRT_SECURE_NO_WARNINGS)
  # WORKAROUND: ros_type_introspection/utils/variant.hpp(235): warning C4267: '=': conversion from 'size_t' to 'uint32_t', possible loss of data
  add_compile_options(/wd4267)
  # WORKAROUND: ros/message_event.h(217): warning C4127: conditional expression is constant
  add_compile_options(/wd4127)
  # WORKAROUND: parson/parson.c(62): warning C4232: nonstandard extension used: 'parson_malloc'
  add_compile_options(/wd4232)
endif()

find_package(catkin REQUIRED COMPONENTS
  roscpp
  std_msgs
  topic_tools
  ros_type_introspection
  dynamic_reconfigure
)

find_package(azure_iot_sdks CONFIG REQUIRED)

add_library(parson STATIC
  parson/parson.c
  parson/parson.h
)

###################################
## catkin specific configuration ##
###################################

catkin_package(
  CATKIN_DEPENDS roscpp std_msgs topic_tools ros_type_introspection
)

###########
## Build ##
###########

include_directories(
  ${catkin_INCLUDE_DIRS}
  ${IOTHUB_CLIENT_INCLUDES}
)

add_executable(${PROJECT_NAME}_node src/ros_azure_iothub_cpp_node.cpp)

link_directories(${catkin_LIBRARY_DIRS})

target_link_libraries(${PROJECT_NAME}_node
  ${catkin_LIBRARIES}
  parson
)

# prefer to use shared library.
if (TARGET iothub_client_dll)
  target_link_libraries(${PROJECT_NAME}_node
    $<TARGET_LINKER_FILE:iothub_client_dll>
  )
else()
  target_link_directories(${PROJECT_NAME}_node
    PRIVATE $<TARGET_LINKER_FILE_DIR:iothub_client>
  )
  target_link_libraries(${PROJECT_NAME}_node
    iothub_client
  )
endif()

#############
## Install ##
#############

install(TARGETS ${PROJECT_NAME}_node
  ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
)

install(DIRECTORY launch/
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}/launch
)

#############
##  Test   ##
#############

if(WIN32)
  if(CATKIN_ENABLE_TESTING)
    find_package(rostest REQUIRED)
    add_rostest_gtest(${PROJECT_NAME}_testnode test/testiothub.launch test/testiothub.cpp)
    target_link_libraries(${PROJECT_NAME}_testnode iothub_service_client ${catkin_LIBRARIES} )
  endif()
endif()
