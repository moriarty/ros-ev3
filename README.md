# ROS on the Lego EV3

At 300Mhz and 16 Mb RAM, the EV3 is a huge step up over the previous Lego NXT.

This project aimed to get ROS running on the EV3, however roscore itself wasn't able to run on the brick directly.

This project provides instructions on how to install ros_comm, the ros communication stack onto the ev3, which allows ROS nodes to run on the EV3 as long as their is a roscore running on another computer on the network.

Latest Status
------

- I haven't had any time to work on this, but due to the increased number of emails and the activity I've seen over that the [ev3dev](http://www.ev3dev.org) project I thought now would be a good time to update these instructions.
- using this [python-ev3 package](https://github.com/topikachu/python-ev3) and a pyhton console I controlled the motors from ros topic input. no video proof yet, it was just a quick hack to test it. 
- rospy talker-subscriber working. If you're looking for proof [here is boring video of my ssh terminals](http://youtu.be/ZgA7DgbuVEs)
- roscore not working on the ev3, this is due to the resources required, but with roscore on the network nodes still work.
- All ROS dependencies have been built and installed.

See [brickstrap-build-status.md](https://github.com/moriarty/ros-ev3/blob/master/brickstrap-build-status.md) for the steps I used.

ToDos/Wishlist:
------

- create an sd image to share ev3dev with ros already included. Not the recommended way to do it but some people my just want to quickly test.
- test roscpp, and all the motors and sensors (currently I've only been testing one motor and one sensor assuming they'll all work.)
- create a [controller](http://wiki.ros.org/ros_control#Overview) for the EV3 which listens to ros topics for motor commands and publishes the encoder ticks
- write a configurable differential driver controller which takes [geometry_msgs/Twist](http://wiki.ros.org/geometry_msgs) to allow ros navigation to control the EV3.
- create publishers for the EV3 sensors
- build debs?
- eventually add a ros tab to brickman for ev3dev with ros. 
  - Set ROS_MASTER_URI 
  - Launch the ros controller node
  - Configuration of the controller node would be done via ssh on a per robot basis, but once the robot is built it would be handy to quickly connect it to ROS without ssh first into the brick and launching
