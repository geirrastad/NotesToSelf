# Setting up ROS2 environment for IAAM

1) Install ROS2 package sources as described in ROS2 Wiki

2) sudo apt install ros-crystal-ros-base ros-crystal-rclcpp ros-crystal-rclpy 

3) pip uninstall em

4) pip install empy catkin_pkg

You may now clone the iaam_ros2 sources and build
NOTE: After cleaning out the build directory, or when builing for the first time, please
run:
```
mkdir build; cd build; cmake -D FRESH_BUILD=true ..; make
cmake ..; make
```
This will create some dependencies for event_mapper.
