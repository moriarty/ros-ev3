# ROS on the Lego EV3

At 300MHz and 64MB RAM, the EV3 is a huge step up over the previous Lego NXT.

This project provides instructions on how to install ros_comm, the ros communication stack onto the EV3, which allows ROS nodes to run on the EV3 as long as their is a roscore running on another computer on the network.

This project aimed to get ROS running on the EV3, however roscore itself isn't able to run on the brick directly.

Big thanks to the work done by those contributing to the [ev3dev project](http://www.ev3dev.org) & their github [issues](https://github.com/ev3dev/ev3dev/issues)

Latest Updates
------

- The instructions have been updated and tested with latested release of [ev3dev](http://www.ev3dev.org) on a virtual machine running Ubuntu 14.04.3, and installing ROS Indigo.
- I have been told via email that the instructions also work for Jade. I haven't tested this, but it should work.
- A user has reported that they were able to follow the same instructions with minimal issues and install ROS Jade. 
- using this [python-ev3 package](https://github.com/topikachu/python-ev3) and a python console I controlled the motors from ros topic input. no video proof yet, it was just a quick hack to test it. 
- rospy talker-subscriber working. If you're looking for proof [here is boring video of my ssh terminals](http://youtu.be/ZgA7DgbuVEs)

How To
-----

See [brickstrap-build-status.md](https://github.com/moriarty/ros-ev3/blob/master/brickstrap-build-status.md) for the steps I used.

See Also
-----
The home of these instructions should eventually be on the official ROS wiki: http://wiki.ros.org/Robots/EV3.
For now, the wiki page just links back to here. However, it is worth noting that there is a another project under development, using Yocto Linux and a ROS Meta Layer, See wiki page for more details (they also link back to github). 

ToDos/Wishlist:
------

- Create a tutorial and example programs to start using ros_comm on the EV3 for users who aren't familiar with ROS.
- Create a tutorial and examples for compiling ROS c++ nodes in brickstrap and transfering them to the EV3.
- Create a script or brickstrap config to simplify the process.
- Create a [controller](http://wiki.ros.org/ros_control#Overview) for the EV3 which listens to ros topics for motor commands and publishes the encoder ticks.
- Write a configurable differential driver controller which takes [geometry_msgs/Twist](http://wiki.ros.org/geometry_msgs) to allow ros navigation to control the EV3.
- Create publishers for the EV3 sensors
- Build debs?
- eventually add a ROS tab to brickman for ev3dev with ros. 
  - Set ROS_MASTER_URI 
  - Launch the ros controller node
  - Configuration of the controller node would be done via ssh on a per robot basis, but once the robot is built it would be handy to quickly connect it to ROS without ssh first into the brick and launching
