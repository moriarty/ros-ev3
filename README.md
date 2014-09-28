ros-ev3
=======

Currently trying to get ros core running on an ev3.

The lego EV3 is a huge improvement in power over the old Lego NXT, and is able to boot linux off of the micro sd card out of the box.

Unfortunately, the AM1808 is an ARM9 chip, using the ARMv5 instruction, and Ubuntu only supports ARMv7 and newer. That would have been too easy. 

I just got my hands on the EV3 last week, and so far have just started by installing debian for ev3 from www.ev3dev.org. 
This README.md is currenly to keep track of my steps, and it will hopefully be cleaned up once roscore can be run on the EV3.

WIFI:
the officially supported wifi dongle, netgear WNA1100 isn't available anymore. I tried my luck with the netgear WNA3300, which didn't work. But following the ev3dev instructions share internet via usb on my Macbook was straight forward. 

Once apt-get upgrade has finished on the EV3, (slow process) I'll try the netgear WNA3300 and ISY IWL 2000 for wireless. 
- the WiFi dongles I have do not work. 


ROS DISTRO
I'm forturnate enough to have 2 EV3s to play with, and will be trying to get both ROS Hydro and Indigo working. 
The idea is to have a working set of instructions to install a linux distro and ROS to a micro SD card.


apt-get install build-essential


alex@ev3dev:~/ros_catkin_ws$ rosdep install --from-paths src --ignore-src --rosdistro indigo -y
ERROR: the following packages/stacks could not have their rosdep keys resolved
to system dependencies:
rosconsole: No definition of [log4cxx] for OS version []
roslisp: No definition of [boost] for OS version []
catkin: No definition of [python-argparse] for OS version []
message_filters: No definition of [boost] for OS version []
roslib: No definition of [boost] for OS version []
rostime: No definition of [boost] for OS version []
rostest: No definition of [boost] for OS version []
rospack: No definition of [boost] for OS version []
cpp_common: No definition of [libconsole-bridge-dev] for OS [debian]
rosbag: No definition of [boost] for OS version []
rosbag_storage: No definition of [libconsole-bridge-dev] for OS [debian]
