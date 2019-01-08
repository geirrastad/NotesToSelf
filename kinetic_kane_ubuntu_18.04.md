# Installing Kinetic Kane on Ubuntu 18.04 LTS

This document will guide you through the process of installing Kinetic Kane from source to your Ubuntu 18.04 LTS. If you are new to ROS, you should probably gor for a pre-built Melodic installation.

A few notes before you embark on this mission:
   The Kinetik Desktop-full installation WILL fail, as I have not bothered to fix it (did not need the extra stuff). There are some dependencies that fail and you will manually have to build the components yourselves.
   Also, I have not done a regression test on ROS. It still may unexpectedly fail. If it do I will make a note of what went wrong here (or possibly a fix).
   

I used the [Installing from source](http://wiki.ros.org/kinetic/Installation/Source) guide, but it will fail if you try this on 18.04. It was made for 16.04. Now let's start!

## 1. First steps
First you must install Ubuntu and download ROS Source installation files

1) [Download and install Ubuntu](https://www.ubuntu.com/download/desktop). Make it a plain installation, and adapt to your language and default settings

2) Run this command to set up Kinetick Source Installation
```
$ sudo apt-get install python-rosdep python-rosinstall-generator python-wstool python-rosinstall build-essential
```

3) Initialize source distro
```
$ sudo rosdep init
$ rosdep update
```

4) Create Kinetic workspace
```
$ mkdir ~/ros_catkin_ws
$ cd ~/ros_catkin_ws
```

5) Fetch the Desktop install:
```
$ rosinstall_generator desktop --rosdistro kinetic --deps --wet-only --tar > kinetic-desktop-wet.rosinstall
$ wstool init -j8 src kinetic-desktop-wet.rosinstall
```

Now, trying to resolve dependencies with:
```
 rosdep install --from-paths src --ignore-src --rosdistro kinetic -y
```
will fail with the following errors:
```
ERROR: the following packages/stacks could not have their rosdep keys resolved
to system dependencies:
rqt_bag_plugins: No definition of [python-imaging] for OS version [bionic]
opencv3: No definition of [libvtk-qt] for OS version [bionic]
rosbag: No definition of [python-imaging] for OS version [bionic]
```

## 2. Fix Kinetic dependencies

We need to fix the two issues from above (plus one hidden that will bite you after the next step)

1) cd to your home directory and Clone the "rosdistro" git:
```
$ cd ~
$ git clone https://github.com/ros/rosdistro
$ cd rosdistro/rosdep
```

2) Edit the base.yaml file. Change this area
```
gazebo7:
  arch: [gazebo]
  debian:
    buster: [gazebo7]
    jessie: [gazebo7]
    stretch: [gazebo7]
  fedora: [gazebo]
  gentoo: [=sci-electronics/gazebo-7*]
  slackware: [gazebo]
  ubuntu: [gazebo7]
```
And change the "**ubuntu: [gazebo7]**" to "**ubuntu: [gazebo9]**" like this:
```
gazebo7:
  arch: [gazebo]
  debian:
    buster: [gazebo7]
    jessie: [gazebo7]
    stretch: [gazebo7]
  fedora: [gazebo]
  gentoo: [=sci-electronics/gazebo-7*]
  slackware: [gazebo]
  ubuntu: [gazebo9]
```
Then look up this part (same file):
```
  ubuntu:
    '*': [libvtk6-dev]
```
and add an line just below like this:
```
  ubuntu:
    '*': [libvtk6-dev]
    bionic: [libvtk6-dev]
```
A few lines down we also need to replace this:
```
  ubuntu:
    '*': [libvtk6-qt-dev]
```
By this:
```
  ubuntu:
    '*': [libvtk6-qt-dev]
    bionic: [libvtk6-qt-dev]
```



save the file!

4) Now edit the python.yaml and locate this area:
```
  ubuntu:
    '*': [python-pil]
    artful: [python-imaging]
```
And add a line like this:
```
  ubuntu:
    '*': [python-pil]
    bionic: [python-pil]
    artful: [python-imaging]
```
The find this area:
```
  ubuntu:
    artful: [python-rosdep]
    bionic: [python-rosdep]
```
And change the bionic entry to read python-rosdep2. Like this:
```
  ubuntu:
    artful: [python-rosdep]
    bionic: [python-rosdep2]
```

Save and exit!

5) Now go here:
```
$ cd /etc/ros/rosdep/sources.list.d
$ sudo nano 20-default.list
```
   
    and comment out two lines in the 20-default.list:
    
```
    #yaml https://raw.githubusercontent.com/ros/rosdistro/master/rosdep/base.yaml
    #yaml https://raw.githubusercontent.com/ros/rosdistro/master/rosdep/python.yaml
```
    
    File content after edit:
    
```
# os-specific listings first
yaml https://raw.githubusercontent.com/ros/rosdistro/master/rosdep/osx-homebrew.yaml osx

# generic
# yaml https://raw.githubusercontent.com/ros/rosdistro/master/rosdep/base.yaml
# yaml https://raw.githubusercontent.com/ros/rosdistro/master/rosdep/python.yaml
yaml https://raw.githubusercontent.com/ros/rosdistro/master/rosdep/ruby.yaml
gbpdistro https://raw.githubusercontent.com/ros/rosdistro/master/releases/fuerte.yaml fuerte

# newer distributions (Groovy, Hydro, ...) must not be listed anymore, they are being fetched from the rosdistro index.yaml instead
```

6) Create a new file '10-mydistro.list' with the following content:
```
yaml file:/<home_dir>/rosdistro/rosdep/base.yaml
yaml file:/<home_dir>/rosdistro/rosdep/python.yaml
```
Remember to replace <home_dir> with your actual home directory!

7) Now you must update ros package source lists
```
$ rosdep update
```

8) You can now check ros dependencies by running:
```
$ cd ~/ros_catkin_ws
$ rosdep install --from-paths src --ignore-src --rosdistro kinetic -y
```

9) It is time to install other dependencies. Run the following commands:
```
$ sudo apt install libopencv-core3.2 libopencv-core-dev libopencv-dev libbullet-dev libtf2-bullet-dev python-pil python-pip gazebo9
```
    
The environment is now prepared for Kinetic source installation.

## 3. Updating Kinetic Kane Sources


1) Some of the downloaded ROS source packages will fail, and we will have to swap with newest git versions
```
$ cd ~
$ git clone https://github.com/ros/rospack
$ git clone https://github.com/ros/geometry2
$ git clone https://github.com/ros/roscpp_core
$ git clone https://github.com/ros/pluginlib
$ git clone https://github.com/ros-visualization/qt_gui_core

$ rm -rf ~/ros_kinetic/src/rospack
$ rm -rf ~/ros_kinetic/src/geometry2
$ rm -rf ~/ros_kinetic/src/roscpp_core
$ rm -rf ~/ros_kinetic/src/pluginlib
$ rm -rf ~/ros_kinetic/src/qt_gui_core
$ mv rospack/ geometry2/ roscpp_core/ ros_kinetic/src/
```
We have now updated these modules to the latest releases. Note: this is mostly the Melodic versions of these modules. 
should try to checkout to kineticcmake 
   
2) Now you need to update some packages. Run:
```
$ sudo pip install --upgrade catkin_pkg_modules
```

3) Then it is time to build Kinetic. This takes some time to finish, so go and make a cup or coffee after running:
```
$ cd ~
$ ./src/catkin/bin/catkin_make_isolated --install -DCMAKE_BUILD_TYPE=Release
```
OR, if you want to put Kinetic in /opt:
```
$ cd ~
$ sudo ./src/catkin/bin/catkin_make_isolated --install -DCMAKE_BUILD_TYPE=Release --install-space /opt/ros/kinetic
```

4) Finally update .basrc to automatically set up your environment (optional):
```
   echo "source ~/ros_kinetic/install_isolated/setup.bash" >> ~/.bashrc
```
OR, if you installed into the /opt directory:
```
   echo "source /opt/ros/kinetic/setup.bash" >> ~/.bashrc
```

## Notes

Following this guide should successfully build all 198 packages in Kinetic.
However, if something crashes during build, please remove (rm -rf) the package directory from both build_isolated and devel_isolated before re-running build. Mysterious Things may happen if you don't.
My machine crashed everytime I used 2 CPUS with 4 cores (I'm building and testing in VMWare Workstation 14). With 1 CPU, 4 cores and 20Gb RAM it worked flawlessly. 


**That's it!. You should now have a functioning Kinetic Kane installation on Ubuntu 18.04 LTS**
