cmake_minimum_required(VERSION 3.8)
project(serial_manager_pkg)

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

# find dependencies
find_package(ament_cmake REQUIRED)
find_package(rclcpp REQUIRED)
find_package(rclcpp_components REQUIRED)
find_package(std_msgs REQUIRED)
find_package(geometry_msgs REQUIRED)
find_package(rosidl_default_generators REQUIRED)

include_directories(
  ${CMAKE_CURRENT_BINARY_DIR}/rosidl_generator_cpp
)

rosidl_generate_interfaces(${PROJECT_NAME}
  "msg/SerialMsg.msg"
  DEPENDENCIES geometry_msgs
)

add_library(serial_manager SHARED
   src/serial_manager.cpp)

ament_target_dependencies(serial_manager
  rclcpp
  rclcpp_components
  std_msgs
)

rclcpp_components_register_node(
    serial_manager
    PLUGIN "serial_manager::SerialManager"
    EXECUTABLE serial_manager_node
)

ament_export_targets(export_component)
install(TARGETS serial_manager
        EXPORT export_component
        ARCHIVE DESTINATION lib
        LIBRARY DESTINATION lib
        RUNTIME DESTINATION bin
)

install(DIRECTORY
  launch/
  DESTINATION share/${PROJECT_NAME}/launch
)

get_default_rmw_implementation(rmw_implementation)
find_package("${rmw_implementation}" REQUIRED)
get_rmw_typesupport(typesupport_impls "${rmw_implementation}" LANGUAGE "cpp")

foreach(typesupport_impl ${typesupport_impls})
  rosidl_get_typesupport_target(cpp_typesupport_target "${PROJECT_NAME}" "${typesupport_impl}")
  target_link_libraries(serial_manager "${cpp_typesupport_target}")
endforeach()

if(BUILD_TESTING)
  find_package(ament_lint_auto REQUIRED)
  # the following line skips the linter which checks for copyrights
  # comment the line when a copyright and license is added to all source files
  set(ament_cmake_copyright_FOUND TRUE)
  # the following line skips cpplint (only works in a git repo)
  # comment the line when this package is in a git repo and when
  # a copyright and license is added to all source files
  set(ament_cmake_cpplint_FOUND TRUE)
  ament_lint_auto_find_test_dependencies()
endif()

ament_package()
