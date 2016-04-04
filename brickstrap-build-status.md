This was a work in progress status report on getting ROS onto ev3dev using brickstrap. Now it's become the main instructions on how to use brickstrap to build the ros_comm packages for ev3-dev

#### These steps have been tested in Ubuntu 14.04.3

##### Install brickstrap

1. Follow the [ev3dev tutorial for installing brickstrap](http://www.ev3dev.org/docs/tutorials/using-brickstrap-to-cross-compile/). 
2. Set up a new brickstrap space to work in:
  
  - the -d option allows you to name this workspace
  - the -b option is for which board your using, ev3-ev3dev-jessie is for the Lego EV3.
  ```
  user@host$ brickstrap -b ev3-ev3dev-jessie -d ev3dev-ros create-rootfs
  ```
  This command takes a few minutes.
  
3. Enter the brickstrap shell:

  ```
  user@host$ brickstrap -b ev3-ev3dev-jessie -d ev3dev-ros shell
  (brickstrap)root@host#
  ```

##### Install ros_comm
Once inside of a brickstrap shell:

1. Install some basics, needed to build and extract further dependencies. Change into host-rootfs (explained in brickstrap tutorial)

  ```
  (brickstrap)root@host# cd /host-rootfs/home/user
  (brickstrap)root@host# apt-get update
  (brickstrap)root@host# apt-get install unzip bzip2 build-essential
  ```

2. Install the ROS system dependencies using ```apt-get install```. <br>
  I've added them in list so that they can be updated and maintained easily. 

  ```
  (brickstrap)root@host# wget https://raw.githubusercontent.com/moriarty/ros-ev3/master/ros-dependencies.debs
  (brickstrap)root@host# bash ros-dependencies.debs
  ```

3. Next install the some python packages available through pip

  ```
  (brickstrap)root@host# pip install -U rosdep rosinstall_generator wstool rosinstall catkin_pkg rospkg
  ```
4. sbcl needs to be downloaded, there is a armel binary available for 1.2.7 <br>
  http://www.sbcl.org/platform-table.html <br>
  Downlad, unpack it, change to the directory and run the install script: <br>

  ```
  (brickstrap)root@host# wget http://netcologne.dl.sourceforge.net/project/sbcl/sbcl/1.2.7/sbcl-1.2.7-armel-linux-binary.tar.bz2
  (brickstrap)root@host# tar -xjf sbcl-1.2.7-armel-linux-binary.tar.bz2
  (brickstrap)root@host# cd sbcl-1.2.7-armel-linux
  (brickstrap)root@host# INSTALL_ROOT=/usr/local sh install.sh
  (brickstrap)root@host# cd ..
  ```

5. Initialize rosdep:

  ```
  (brickstrap)root@host# rosdep init
  ```
  
6. Debian jessie is not officially supported by ROS, so we need to change where it will look for some packages<br>
  Open the 20-default.list file:
  
  ```
  (brickstrap)root@host# nano /etc/ros/rosdep/sources.list.d/20-default.list
  ```
  
  Add the following line to the beginning, in the os-specific listing section.
  ```
  # os-specific listings first
  yaml https://raw.githubusercontent.com/moriarty/ros-ev3/master/ev3dev.yaml
  ```
  
  update rosdep. 
  ```
  (brickstrap)root@host# rosdep update
  ```

7. I created and changed to a new directory ros_comm, just to keep things organized. <br>
  Then create the rosinstall file, initialize the ros workspace, and check the ros dependencies are all met.
  Note: only ros_comm and common_msgs will be installed, if you need more, add them to rosinstall_generator command.

  ```
  (brickstrap)root@host# mkdir ros_comm && cd ros_comm
  (brickstrap)root@host# rosinstall_generator ros_comm common_msgs --rosdistro indigo --deps --wet-only --tar > indigo-ros_comm-wet.rosinstall
  (brickstrap)root@host# wstool init src indigo-ros_comm-wet.rosinstall
  (brickstrap)root@host# rosdep check --from-paths src --ignore-src --rosdistro indigo -y --os=debian:jessie
  ```

8. It's time to install ros using catkin_make_isolated.

  ```
  (brickstrap)root@host# ./src/catkin/bin/catkin_make_isolated --install --install-space /opt/ros/indigo -DCMAKE_BUILD_TYPE=Release
  ```
  
  This step might take a while. I'm running the brickstrap shell inside of a Virtual Ubuntu 14.04 inside of OSX 10.10, not the ideal setup for speed, but I'm unable to upgrade to 14.04 on my main linux partition. I've given the VM 4 of 8 cores and 4Gb of ram, With this configuration the process took almost exactly one hour. 

9. Exit the brickstrap shell and create a tar of the brickstrap rootfs and a disk image from the tar.
   The python bindings for ev3dev are available by default. If you need c++, see section below and return to this step later.
  
  ```
  (brickstrap)root@host# exit
  user@host$ brickstrap -d ev3dev-ros -f create-tar
  user@host$ brickstrap -d ev3dev-ros -f create-image
  ```
  

10. Create an SD card from the image, there are two ways to do this:<br>
  - [gui](http://www.ev3dev.org/docs/tutorials/writing-sd-card-image-ubuntu-disk-image-writer/)
  - [cli](http://www.ev3dev.org/docs/tutorials/writing-sd-card-image-linux-command-line/)

##### Install ev3dev-lang-cpp

[ev3dev-lang-cpp](https://github.com/ddemidov/ev3dev-lang-cpp) is the c++ language bindings for ev3dev. Here are unofficial installation instructions. Run these instructions inside of the brickstrap shell from above. If you've closed your brickstrap shell, install brickstrap step 3 will open a new one.

1. Clone ev3dev-lang-cpp

   ```
   (brickstrap)root@host# git clone https://github.com/ddemidov/ev3dev-lang-cpp
   ```
2. Patch the CMakeLists.txt. By default ev3dev-lang-cpp creates a static library which is not installed.
   This patch creates a shared library and adds install for the library and header file.

   ```
   (brickstrap)root@host# cd ev3dev-lang-cpp
   (brickstrap)root@host# wget https://raw.githubusercontent.com/moriarty/ros-ev3/master/install_ev3dev_shared_library_CMakeLists.patch
   (brickstrap)root@host# git apply install_shard_library_CMakeLists.patch
   ```

3. Build and install.

   ```
   (brickstrap)root@host# mkdir build
   (brickstrap)root@host# cd build
   (brickstrap)root@host# cmake ..
   (brickstrap)root@host# make
   (brickstrap)root@host# make install
   ```

4. Return to install ros_comm step 9. 

#### Notes

Don't forget to source setup.bash and export ROS_MASTER_URI.

TODO: test ```export ROS_LANG_DISABLE=genlisp``` to speedup catkin_make_isolated

ROSCORE is not working on the ev3. I don't know if it's possible or when I will have a chance to investigate further. 

Running roscore on a laptop, I was able to run the rospy tutorials talker and listener on the ev3. <br>
I've uploaded a video [here](http://youtu.be/ZgA7DgbuVEs) of the talker/listener running.

I set all the ip addresses of the machines and ev3 in /etc/hosts, might be my own networking issue.


##### OLD patches

I'm updating this guide, but can't afford to install a clean VM. Will move patches that may be fixed to here until I can check on a fresh Ubuntu 14.04 VM that these have indeed been solved. 

1. brickstrap:
  
    at one point before 0.4.0 required a quick fix:
    See this [ev3dev/brickstrap Issue #9](https://github.com/ev3dev/brickstrap/issues/9) <br>
    Replace the preinst.blacklist file
    
    ```
    user@host$ cd /usr/share/brickstrap/ev3-ev3dev-jessie/
    user@host$ sudo rm preinst.blacklist
    user@host$ sudo wget https://raw.githubusercontent.com/ev3dev/brickstrap/master/ev3-ev3dev-jessie/preinst.blacklist 
    ```

2. Multistrap:
  
    The ev3dev developers maintain a patched version of multistrap in their debian mirror. So this will only effect users who had a previous version of multistrap.
    
    There is also an issue on launchpad titled [Multistrap is broken in  14.04](https://bugs.launchpad.net/ubuntu/+source/multistrap/+bug/1313787). In that issue, someone suggests: 

  > Workaround until fix is released: edit /usr/sbin/multistrap and remove $forceyes on line 989.
