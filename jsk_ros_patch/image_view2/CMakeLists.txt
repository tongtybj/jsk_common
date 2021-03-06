# http://ros.org/doc/groovy/api/catkin/html/user_guide/supposed.html
cmake_minimum_required(VERSION 2.8.3)
project(image_view2)

# Use ccache if installed to make it fast to generate object files
find_program(CCACHE_FOUND ccache)
if(CCACHE_FOUND)
  set_property(GLOBAL PROPERTY RULE_LAUNCH_COMPILE ccache)
  set_property(GLOBAL PROPERTY RULE_LAUNCH_LINK ccache)
endif(CCACHE_FOUND)

find_package(catkin REQUIRED COMPONENTS roscpp dynamic_reconfigure  cv_bridge std_msgs sensor_msgs geometry_msgs image_transport tf image_geometry message_filters message_generation std_srvs)

generate_dynamic_reconfigure_options(
  cfg/ImageView2.cfg
)

add_message_files(
  FILES ImageMarker2.msg PointArrayStamped.msg MouseEvent.msg
)

add_service_files(
  FILES ChangeMode.srv
  )

include_directories(include ${catkin_INCLUDE_DIRS})

# Image viewers
find_package(OpenCV 3.1 REQUIRED)
add_executable(image_view2 image_view2.cpp image_view2_node.cpp)
target_link_libraries(image_view2 ${OpenCV_LIBS} ${catkin_LIBRARIES})
add_dependencies(image_view2 ${PROJECT_NAME}_gencpp ${PROJECT_NAME}_gencfg)

# Point Rectangle Extractor
add_executable(points_rectangle_extractor points_rectangle_extractor.cpp)
find_package(Boost REQUIRED COMPONENTS signals)
include_directories(${Boost_INCLUDE_DIRS})
find_package(PCL REQUIRED)
include_directories(${PCL_INCLUDE_DIRS})
target_link_libraries(points_rectangle_extractor ${Boost_LIBRARIES} ${catkin_LIBRARIES} ${OpenCV_LIBS})
add_dependencies(points_rectangle_extractor ${PROJECT_NAME}_gencpp)

generate_messages(DEPENDENCIES geometry_msgs std_msgs)

catkin_package(
    DEPENDS OpenCV PCL
    CATKIN_DEPENDS roscpp cv_bridge std_msgs sensor_msgs geometry_msgs image_transport tf image_geometry message_filters
    INCLUDE_DIRS # TODO include
    LIBRARIES # TODO
)

install(DIRECTORY test
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}
  USE_SOURCE_PERMISSIONS
)
install(TARGETS image_view2 points_rectangle_extractor
  ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
)

if(CATKIN_ENABLE_TESTING)
  find_package(rostest REQUIRED)
  execute_process(
    COMMAND python -c "import sys; sys.exit(ord('$ENV{ROS_DISTRO}'[0]) >= ord('indigo'[0]))"
    RESULT_VARIABLE DISTRO_GE_INDIGO
  )
  if(${DISTRO_GE_INDIGO})
    # FIXME: jsk_tools/test_topic_published.py does not work on hydro travis/jenkins
    # https://github.com/jsk-ros-pkg/jsk_common/pull/1293#issuecomment-164158260
    add_rostest(test/rectangle_mouse_event.launch)
  endif()
endif()
