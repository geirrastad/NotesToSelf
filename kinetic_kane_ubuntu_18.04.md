# Installing Kinetic Kane on Ubuntu 18.04 LTS

This document will guide you through the process of installing Kinetic Kane from source to your Ubuntu 18.04 LTS. If you are new to ROS, you should probably gor for a pre-built Melodic installation.

I used the [Installing from source](http://wiki.ros.org/kinetic/Installation/Source) guide, but it will fail if you try this on 18.04. It was made for 16.04. Now let's start!

## Getting ready
1) [Download and install Ubuntu](https://www.ubuntu.com/download/desktop). Make it a plain installation, and adapt to your language and default settings

2) sudo apt-get install python-rosdep python-rosinstall-generator python-wstool python-rosinstall build-essential

3) sudo rosdep init (you do not need to run rosdep update after this command)

4) cd to your home directory and Clone the "rosdistro" git:
    git clone https://github.com/ros/rosdistro

5) cd rosdistro/rosdep

6) Edit the base.yaml file. Change this area
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
The look up this part (same file):
```
  ubuntu:
    '*': [libvtk6-java]
```
and add an line just below like this:
```
  ubuntu:
    '*': [libvtk6-java]
    bionic: [libvtk6-java]
```
save the file!

7) Now edit the python.yaml and locate this area:
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
Save and exit!

8) Now go here:
    cd /etc/ros/rosdep/sources.list.d
    
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

9) Create a new file '10-mydistro.list' with the following content:
```
yaml file://<home_dir>/rosdistro/rosdep/base.yaml
yaml file://<home_dir>/rosdistro/rosdep/python.yaml
```
Remember to replace <home_dir> with your actual home directory!

10) run "rosdep update"


    
    

2) mkdir <home_dir>/ros_kinetic  (replace all instances of <home_dir> with your home directory path)

