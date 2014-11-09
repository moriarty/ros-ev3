This is a work in progress status report on getting ROS onto ev3dev using brickstrap.

Follow the [instructions](https://github.com/ev3dev/ev3dev/wiki/Using-brickstrap-to-cross-compile-and-debug) here to get brickstrap.
  - I'm currently doing everything inside of the brickstrap shell, and then copying to the ev3.

Once inside of a brickstrap shell:

1. we install the system dependencies using ```sudo apt-get install```. <br>
  I've added them in list so that they can be updated and maintained easily. 

  ```
  user@host$ ./ros-dependencies.debs
  ```
2. Next install the some python packages available through pip

  ```
  pip install -U rosdep rosinstall_generator wstool rosinstall catkin_pkg rospkg
  ```
3. sbcl needs to be downloaded, there is a armel binary available for 1.2.1 <br>
  http://www.sbcl.org/platform-table.html <br>
  unpack it, change to the directory and run: <br>

  ```
  user@host# INSTALL_ROOT=/usr/local sh install.sh
  ```

4. Initialize and update rosdep:

  ```
  user@host# rosdep init
  user@host# rosdep update
  ```
  
5. debian jessie is not officially supported by ros, so now we need to add in some custom rosdep rules.<br>
  create a file ev3dev.yaml and put in it the following.<br>
  Note: I didn't try what would happen if I left these blank, and right now this is a bit of a hack.<br>
  We installed all of these packages in the steps above, so we're just telling rosdep to look for other packages we know are there.

  ```
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
  ```

6. Now we tell rosdep about the file we just created.

  ```
  user@host# vi /etc/ros/rosdep/sources.list.d/20-default.list
  ```
  
  add the following lines to the beginning. update the path to the yaml which you create in the last step. 
  ```
  # for ROS on the ev3 as user "alex"
  # yaml file:///home/alex/ev3dev.yaml
  # for brickstrap the line is:
  yaml file:///host-rootfs/home/alex/workspace/ev3/ev3dev-ros/ev3dev.yaml
  ```

7. run rosdep update again. 

  ```
  user@host# rosdep update
  ```

8. I created a isolated catkin workspace which I later compressed and copied to my user directory once I had the new image of ev3dev with the ros dependencies onto the ev3. <br>
  TODO: put ros into the standard /opt/ros/ location. <br>

  In my ros_catkin_ws, still inside of a brickstrap shell I ran:
  ```
  user@host# rosinstall_generator ros_comm --rosdistro indigo --deps --wet-only --tar > indigo-ros_comm-wet.rosinstall
  user@host# wstool init -j8 src indigo-ros_comm-wet.rosinstall
  ```

9. The ```ros-dependencies.debs``` script was created by trial and error. Hopefully it's still up to date and all the system dependencies are satisifed, but it's a good idea to check first. 

  ```
  root@ubuntu# rosdep check --from-paths src --ignore-src --rosdistro indigo -y --os=debian:jessie
  
  All system dependencies have been satisified
  ```

10. It's time to install for ```catkin_make_isolated``` install. <br>
  from inside the ros_caktin_workspace run:
  ```
  ./src/catkin/bin/catkin_make_isolated --install -DCMAKE_BUILD_TYPE=Release
  ```
  
  This might take a while. On the EV3 itself it took 15+ hours... and failed. <br>
  I'm now running Ubuntu 14.04.1 inside VMWareFusion, and then brickstrap / qemu inside of that. <br> 
  I left it alone for at least an hour. 

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


