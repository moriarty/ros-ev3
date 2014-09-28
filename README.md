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

ROS DISTRO
I'm forturnate enough to have 2 EV3s to play with, and will be trying to get both ROS Hydro and Indigo working. 
The idea is to have a working set of instructions to install a linux distro and ROS to a micro SD card.
