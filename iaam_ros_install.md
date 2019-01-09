# Installing IAAM modules

## 1. FLIR Camera support

1.1) Get latest Spinnaker drivers (spinnaker-1.18.0.17-amd64-pkg.tar.gz). Untar and install:
```
$ cd ~
$ tar -xzvf spinnaker-1.18.0.17-amd64-pkg.tar.gz
$ cd spinnaker-1.18.0.17-amd64
$ sudo ./install_spinnaker.sh
```
Answer "Y" to all quiestions, and when asked for a udev user, enter your user name

1.2) Update GRUB with USB buffers.
     FLIR USB 3.1 (Gen1) cameras needs an extended buffer to work correctly.
     If you intend to use GigE cameras you may skip this, and go straight to chapter 2
     You must add this to your /etc/defaults/grub file like this:
```
$ sudo nano /etc/defaults/grub
```
At the end of the 'GRUB_CMDLINE_LINUX_DEFAULT=' line add usbcore.usbfs_memory_mb=1000 just before the ".

Example file after editing (note: your CMDLINE may vary):
```
GRUB_CMDLINE_LINUX_DEFAULT="quiet usbcore.usbfs_memory_mb=1000"
```
Save the file, the execute:
```
$ sudo update-grub
```
Now reboot!

## 2. Install IAAM dependencies

2.1) External dependencies
```
$ sudo apt install libfltk1.3 libconfig++9v5 ibfltk-cairo1.3 libfltk-forms1.3 libfltk-images1.3 libfltk-gl1.3 libmgl-fltk7.5.0 libopencv-core3.2
```

2.2) IAAM libraries

You should get the latest version of IAAM core libraries (iaam-0.2.0.tgz). The apply this:

```
$ cd /
$ sudo tar -xzvf <path_to_iaam_lib>/iaam-0.2.0.tgz
$ sudo ldconfig
$ cd
```
I will make an installer for IAAM core later ....

## 3. Install IAAM Ros modules

Now it is time to install the ROS modules. These modules are currently built usin Melodic, but they run fine on Kinetic as well
Get the file: iaam_ros_modules.tgz, then:
```
$ mkdir ~/iaam_ros
$ cd ~/iaam_ros
$ tar -xzvf <path_to_iaam_ros_modules.tgz>/iaam_ros_modules.tgz
```

NOTE: If you are running this on Kinetic, you will have to edit the install/_setup_util.py
Locate the line:
```
CMAKE_PREFIX_PATH = '/opt/ros/melodic'.split(';')
```
and change it to:
```
CMAKE_PREFIX_PATH = '/opt/ros/kinetic'.split(';')
```


## 4. Running IAAM Ros

Please change config files to point to the correct camera serial, and your desired frame rates, thresholds etc.
Config files are here:
```
~/iaam_ros/install/config
```

If you are running Melodic you can start by running:
```
$ source install/setup.bash
$ roslaunch install/share/iaam/iaam.launch
```

or creating a new console for each of the modules and running:
```
$ rosrun <package> <package>_node
```

There are five modules:

 * Commander (rosrun commander commander_node)
 * Debug Window (rosrun debug_window debug_window_node)
 * Stream Recorder (rosrun stream_recorder stream_recorder_node)
 * Settings Finder (rosrun settings_finder settings_finder_node)
 * Frame Collector (rosrun frame_collector frame_collector_node)
 
You should start the Frame Collector last, the others can start at your liking.







     
