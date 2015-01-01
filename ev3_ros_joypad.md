This is just some rough notes because [issue 237](https://github.com/ev3dev/ev3dev/issues/237) on the ev3dev project reminded me of something worth sharing.
I started to write this as a comment on the issue, but it's rather off topic so I just put it here. 

I have used a logitech USB joypad to control the ev3 motors. 

With a few rather LARGE caveats, unless you've heard of ROS. I mention this because ros_joy package supports the ps3 joypad as well as the wiimote, xbox joypad and logitech joypads. 

It's probably a lot of overhead to get ros up and running, and familiarize yourself with it. But you could also look into their [ps3 driver code](https://github.com/ros-drivers/joystick_drivers) on github.

- If you're still interested, here is what I did. I apologize in advance that I don't have any python scripts to share it was just hacking on my desktop and I'm away from home. 
- If you're familiar with ROS, the longest step is part 1, but I've created easy to follow instructions.
- If you're not familiar with ROS, then getting it installed on your system and reading through the beginner tutorials takes a day. 

First the setup:

1. I have installed [ROS (Robot Operating System)](http://www.ros.org) onto my ev3, instructions linked to on the [ev3dev.org projects page](http://www.ev3dev.org/projects/)
2. The USB joypad was plugged into my desktop.
3. The EV3 was connected to my network via USB wifi adapter 
4. A python script (ros node) was running on the ev3 
5. A pyhton script (ros node) was running on the desktop
6. I used the existing ros [joy package](http://wiki.ros.org/joy) to deal with the joypad
7. roscore can only run on the desktop, with network set up so that the desktop and ev3 could communicate via hostnames, and the desktop and ev3 had their ROS_MASTER_URI environement variables set to the desktop.  [More notes on this here](https://github.com/moriarty/ros-ev3/blob/master/brickstrap-build-status.md)

The process:

1. run roscore on the desktop
2. launch joypad node on the desktop (this is a default node which only needs configuration) it listens to the joypad and publishes "messages" to a topic /joy (some arrays of button values)
3. wrote a node on the desktop to listen to the /joy topic and publish to /cmd_vel (a std_msgs/Twist)
4. wrote a node on the ev3 to listen to /cmd_vel and command the motors. 


As I was only testing I was just using one motor and going forward or backward. 
But the end idea is to use ros_control and have differential drive controller which publishes wheel velocities to the ev3. 
I'm looking forward to working on the project again soon - but currently working on a something with a sooner deadline. 

I'm also in the mountains with a very unreliable internet connection for the next few days so if you do opt to try ROS I won't be actively available on github. 

If I wanted to use a joypad without ROS, then I would probably start by looking into the ros joypad driver code and see if I could get it working sans-ROS. 

This will be one of the first basic things to get up and running once I'm working on this ros-ev3 project again, so hopefully I'll update this process with more details in a few weeks. 



Note to self: 
- /joy -> ev3dev_teleop: A teleop node to listen to the joypad data.
- /cmd_vel -> ev3dev_driver: A driver node to convert a std_msgs/Twist according to the robot setup (diff, omni)
- ev3dev_driver will publish according to setup motor velocities.
- /ev3/motor_vel/[A-D] -> small script running on ev3 listens for motor velocities and publishes them to the motors
