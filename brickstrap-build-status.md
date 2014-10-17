getting ROS on to the EV3 will require building the packages off the EV3 luckily there is a cool way that makes this easy.

Follow the instructions here to get brickstrap.
https://github.com/ev3dev/ev3dev/wiki/Using-brickstrap-to-cross-compile-and-debug

This is just a place for me to dump some copy and paste stuff, I'll make a how to later.

I greped my command history to get the list of ros dependencies I installed on the EV3. 
I added gdb and python-all-dev because they were suggested and I think they might help. 

    apt-get install build-essential cmake python-pip apt-utils libyaml-dev python-yaml libyaml-cpp-dev libboost-dev liblog4cxx10 liblog4cxx10-dev initramfs-tools libboost-all-dev libc6-dev libpython2.7-stdlib libconsole-bridge-dev libbz2-dev python-coverage liblz4-dev python-paramiko python-nose python-all-dev gdb 
    pip install -U rosdep rosinstall_generator wstool rosinstall catkin_pkg rospkg

sbcl needs to be downloaded, there is a armel binary available for 1.2.1
http://www.sbcl.org/platform-table.html

direct link: http://downloads.sourceforge.net/project/sbcl/sbcl/1.2.1/sbcl-1.2.1-armel-linux-binary.tar.bz2?r=&ts=1413492381&use_mirror=netcologne
unpack it, change to the directory and run:
INSTALL_ROOT=/usr/local sh install.sh

rosdep init
rosdep update

I made a directory ~/workspace/ev3/ev3dev-ros where I will be working on the ev3dev-ros stuff.
Inside I made a ros_catkin_ws directory.

vi /etc/ros/rosdep/sources.list.d/20-default.list 

add a line to the beginning: 
# on the ev3 I added
yaml file:///home/alex/ev3dev.yaml
# in brickstrap the line is:
yaml file:///host-rootfs/home/alex/workspace/ev3/ev3dev-ros/ev3dev.yaml

The file contains:
alex@ev3dev:/root$ cat ~/ev3dev.yaml 
libconsole-bridge-dev:
  debian:
    jessie: [libconsole-bridge-dev]
python-rosdep:
  debian:
    jessie: [python]
sbcl:
  debian:
    jessie: [libc6]
python-rospkg:
  debian:
    jessie: [python]
python-catkin-pkg:
  debian:
    jessie: [python]

run rosdep update again
rosdep update

In my ros_catkin_ws I ran:
rosinstall_generator ros_comm --rosdistro indigo --deps --wet-only --tar > indigo-ros_comm-wet.rosinstall
wstool init -j8 src indigo-ros_comm-wet.rosinstall

root@ubuntu:/host-rootfs/home/alex/workspace/ev3/ev3dev-ros/ros_catkin_ws# rosdep check --from-paths src --ignore-src --rosdistro indigo -y --os=debian:jessie
System dependencies have not been satisified:
apt	python-mock
apt	python-empy
apt	python-netifaces
apt	libtinyxml-dev
apt	python-numpy
apt	python-imaging
apt	libgtest-dev

Fix those:
root@ubuntu:/host-rootfs/home/alex/workspace/ev3/ev3dev-ros/ros_catkin_ws# apt-get install python-mock python-empy python-netifaces libtinyxml-dev python-numpy python-imaging libgtest-dev

Check again:
root@ubuntu:/host-rootfs/home/alex/workspace/ev3/ev3dev-ros/ros_catkin_ws# rosdep check --from-paths src --ignore-src --rosdistro indigo -y --os=debian:jessie
All system dependencies have been satisified
root@ubuntu:/host-rootfs/home/alex/workspace/ev3/ev3dev-ros/ros_catkin_ws# rosdep install --from-paths src --ignore-src --rosdistro indigo -y --os=debian:jessie
#All required rosdeps installed successfully

Then run:
./src/catkin/bin/catkin_make_isolated --install -DCMAKE_BUILD_TYPE=Release

This might take a while. On the EV3 itself it took 15+ hours... and failed. 
I'm now running Ubuntu 14.04.1 inside VMWareFusion, and then brickstrap / qemu inside of that. 
7 of 49 packages have been made in 5 minutes. 


The first try died on 9 of 49. my fault. The virtual machine display went to sleep. 
running again got to 20 of 49 very quickly. 

Everything finished. created a tar of my ros_catkin_ws and moved it to the EV3

source ~/ros_catkin_ws/install_isolated/setup.bash

alex@ev3dev:~$ export ROS_MASTER_URI=http://localhost:11311

alex@ev3dev:~$ roscore
... logging to /home/alex/.ros/log/6615ed02-5590-11e4-a515-021653462e42/roslaunch-ev3dev-1127.log
Checking log directory for disk usage. This may take awhile.
Press Ctrl-C to interrupt
Done checking log file disk usage. Usage is <1GB.

started roslaunch server http://ev3dev:51216/
ros_comm version 1.11.9


SUMMARY
========

PARAMETERS
 * /rosdistro: indigo
 * /rosversion: 1.11.9

NODES

auto-starting new master
process[master]: started with pid [1144]
ERROR: could not contact master [http://ev3dev:11311/]
The traceback for the exception was written to the log file
[master] killing on exit
Traceback (most recent call last):
.... and a bunch of crap


