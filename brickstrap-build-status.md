This is a work in progress status report on getting ROS onto ev3dev using brickstrap.

Follow the [instructions](https://github.com/ev3dev/ev3dev/wiki/Using-brickstrap-to-cross-compile-and-debug) here to get brickstrap. But note:
  - ```apt-get install brickstrap``` doesn't include a recent patch. See this [ev3dev Issue #190](https://github.com/ev3dev/ev3dev/issues/190) <br>
    I checked out the source code and replaced the ev3dev-jessie brickstrap uses with the new one.
    
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
  root@host# vi /etc/ros/rosdep/sources.list.d/20-default.list
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
  root@host# rosdep update
  ```

8. I created a isolated catkin workspace which I later compressed and copied to my user directory once I had the new image of ev3dev with the ros dependencies onto the ev3. <br>
  TODO: put ros into the standard /opt/ros/ location. There is a note on this [here](http://wiki.ros.org/hydro/Installation/Source#Maintaining_a_Source_Checkout) <br>

  In my ros_catkin_ws, still inside of a brickstrap shell I ran:
  ```
  root@host# rosinstall_generator ros_comm --rosdistro indigo --deps --wet-only --tar > indigo-ros_comm-wet.rosinstall
  root@host# wstool init -j8 src indigo-ros_comm-wet.rosinstall
  ```

9. The ```ros-dependencies.debs``` script was created by trial and error. Hopefully it's still up to date and all the system dependencies are satisifed, but it's a good idea to check first. 

  ```
  root@host# rosdep check --from-paths src --ignore-src --rosdistro indigo -y --os=debian:jessie
  
  All system dependencies have been satisified
  ```

10. It's time to install for ```catkin_make_isolated``` install. <br>
  from inside the ros_caktin_workspace run:
  ```
  ./src/catkin/bin/catkin_make_isolated --install -DCMAKE_BUILD_TYPE=Release
  ```
  
  This might take a while. On the EV3 itself it took 15+ hours... and failed. <br>
  I'm now running Ubuntu 14.04.1 inside VMWareFusion, and then brickstrap / qemu inside of that. <br> 
  I left it alone for an hour and (after a few failed attempts) it was done when I returned. 


#### Notes

Once Everything finished. I created a tar of my ros_catkin_ws and moved it to my home directory on the EV3.<br>
Don't forget to source setup.bash and export ROS_MASTER_URI.


ROSCORE is not working on the ev3. I don't know if it's possible or when I will have a chance to investigate further. 

Running roscore on a laptop, I was able to run the rospy tutorials talker and listener on the ev3. <br>
I've uploaded a video [here](http://youtu.be/ZgA7DgbuVEs) of the talker/listener running.

I set all the ip addresses of the machines and ev3 in /etc/hosts, might be my own networking issue.


