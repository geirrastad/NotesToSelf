# Setting up ROS2 environment for IAAM

1) Install ROS2 package sources as described in ROS2 Wiki

2) sudo apt install ros-crystal-ros-base ros-crystal-rclcpp ros-crystal-rclpy 

3) pip uninstall em

4) pip install empy catkin_pkg

You may now clone the iaam_ros2 sources and build
NOTE: in src/CMakeLists.cmake you must comment out add_subdirectory(event_mapper) in first run!
This should have been a separate project... time ... no time ...
