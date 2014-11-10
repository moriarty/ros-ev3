This is a work in progress status report on getting ROS onto ev3dev using brickstrap.

Follow the [instructions](https://github.com/ev3dev/ev3dev/wiki/Using-brickstrap-to-cross-compile-and-debug) here to get brickstrap. But note:
  - ```apt-get install brickstrap``` doesn't include a recent patch. See this [ev3dev Issue #190](https://github.com/ev3dev/ev3dev/issues/190) <br>
    I checked out the source code and replaced the ev3dev-jessie brickstrap uses with the new one.<br>
    We will need to modify this again once more before we create the image. 
    
    ```
    user@host$ git checkout git@github.com:ev3dev/brickstrap
    user@host$ sudo rm -rf /usr/share/brickstrap/ev3dev-jessie
    user@host$ sudo mv ev3dev-jessie /usr/share/brickstrap/ev3dev-jessie
    ```

Once you've updated the ev3dev-jessie you can run

```
user@host$ brickstrap -b ev3dev-jessie -d ev3dev-ros all
```
This command takes a few minutes. When it's finished you can enter the brickstrap shell.

```
user@host$ brickstrap -b ev3dev-jessie -d ev3dev-ros shell
root@host#
```

Once inside of a brickstrap shell:

1. Change into the host-rootfs as suggested in guide above. <br>
  Then install the system dependencies using ```apt-get install```. <br>
  I've added them in list so that they can be updated and maintained easily. 

  ```
  root@host# cd /host-rootfs/home/user/workspace/ev3
  root@host# apt-get update
  root@host# wget https://raw.githubusercontent.com/moriarty/ros-ev3/master/ros-dependencies.debs
  root@host# chmod +x ros-dependencies.debs
  root@host# ./ros-dependencies.debs
  ```

2. Next install the some python packages available through pip

  ```
  root@host# pip install -U rosdep rosinstall_generator wstool rosinstall catkin_pkg rospkg
  ```
3. sbcl needs to be downloaded, there is a armel binary available for 1.2.1 <br>
  http://www.sbcl.org/platform-table.html <br>
  Downlad, unpack it, change to the directory and run the install script: <br>

  ```
  root@host# wget http://netcologne.dl.sourceforge.net/project/sbcl/sbcl/1.2.1/sbcl-1.2.1-armel-linux-binary.tar.bz2
  root@host# tar -xjf sbcl-1.2.1-armel-linux-binary.tar.bz2 
  root@host# INSTALL_ROOT=/usr/local sh install.sh
  ```

4. Initialize and update rosdep:

  ```
  root@host# rosdep init
  root@host# rosdep update 
  ```
  
  Note: the rosdep update may not be needed.
  Ignore the following warning, fix it later. 
  Warning: running 'rosdep update' as root is not recommended.
  You should run 'sudo rosdep fix-permissions' and invoke 'rosdep update' again without sudo.

  
5. debian jessie is not officially supported by ros, so now we need to add a custom ev3dev.yaml<br>
  And add this file to the rosdep sources.
  
  ```
  root@host# wget https://raw.githubusercontent.com/moriarty/ros-ev3/master/ev3dev.yaml
  root@host# vi /etc/ros/rosdep/sources.list.d/20-default.list
  ```
  
  Add the following lines to the beginning, update the path to where wget just put the file. 
  ```
  # for ROS on the ev3 as user "alex"
  # yaml file:///home/alex/ev3dev.yaml
  # for brickstrap the line is:
  yaml file:///host-rootfs/home/alex/workspace/ev3/ev3dev-ros/ev3dev.yaml
  ```
  
  update rosdep again. 
  ```
  root@host# rosdep update
  ```

6. I created and changed to a new directory ros_comm, just to keep things organized. <br>
  Then create the rosinstall file, initialize the ros workspace, and check the ros dependencies are all met.

  ```
  root@host# mkdir ros_comm && cd ros_com
  root@host# rosinstall_generator ros_comm --rosdistro indigo --deps --wet-only --tar > indigo-ros_comm-wet.rosinstall
  root@host# wstool init -j8 src indigo-ros_comm-wet.rosinstall
  root@host# rosdep check --from-paths src --ignore-src --rosdistro indigo -y --os=debian:jessie
  ```

7. It's time to install ros using catkin_make_isolated.

  ```
  root@host# ./src/catkin/bin/catkin_make_isolated --install --install-space /opt/ros/indigo -DCMAKE_BUILD_TYPE=Release
  ```
  
  This step might take a while. I'm running the brickstrap shell inside of a Virtual Ubuntu 14.04 inside of OSX 10.10, not the ideal setup for speed, but I'm unable to upgrade to 14.04 on my main linux partition. I've given the VM 4 of 8 cores and 4Gb of ram, With this configuration the process took almost exactly one hour. 

8. Exit the brickstrap shell and create a tar of the brickstrap rootfs and a disk image from the tar.
  
  brickstrap is using the ev3dev-jessie "BOARD" settings which create a 900M Image. After installing ROS the tar is 1.3G
  I increased it to 1500M. 

  ```
  root@host# exit
  user@host# sudo vi /usr/share/brickstrap/ev3dev-jessie/config
  ```
  
  Change ```IMAGE_FILE_SIZE="900M"``` to ```IMAGE_FILE_SIZE="1500M"```
  
  ```
  user@host$ brickstrap -d ev3dev-ros -f create-tar
  user@host$ brickstrap -d ev3dev-ros -f create-image
  ```
  

9. Create an SD card from the image, there are two ways to do this:<br>
  - [gui](http://www.ev3dev.org/docs/tutorials/writing-sd-card-image-ubuntu-disk-image-writer/)
  - [cli](http://www.ev3dev.org/docs/tutorials/writing-sd-card-image-linux-command-line/)


#### Notes

Don't forget to source setup.bash and export ROS_MASTER_URI.

ROSCORE is not working on the ev3. I don't know if it's possible or when I will have a chance to investigate further. 

Running roscore on a laptop, I was able to run the rospy tutorials talker and listener on the ev3. <br>
I've uploaded a video [here](http://youtu.be/ZgA7DgbuVEs) of the talker/listener running.

I set all the ip addresses of the machines and ev3 in /etc/hosts, might be my own networking issue.


