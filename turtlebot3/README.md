### this is a log for working with the rasp pi b+ for turtlebot3

#### I brought a pi3B+ home to test
The goal is to setup turtlebot3 with ROS Melodic Navigation
This follows the Robotis EManual here (https://emanual.robotis.com/docs/en/platform/turtlebot3/overview/)
but I have made several modifications to stream line the procedure

##### Graphical Install 
First you can try to install through the Mate desktop environment. This did not work for me:

1) Install Ubuntu MATE on TurtleBot PC 
* download Mate 18.04 64bit image for pi 3B + from Ubuntu Mate website (https://ubuntu-mate.org/download/arm64/)
* use pi-imager to load the image to SD card (https://www.raspberrypi.org/downloads/) 
* insert SD card and boot pi
* open MATE desktop and it seems to work, graphics are terribly slow
* test the internet

2) Install ROS on TurtleBot PC - Robotis Emanual calls the 'manual', do not use their script without invesitigating first
* install ros successfully and test roscore
* freezing and crashing seems to be a common issue when using the desktop, some have suggested trying the 32bit option
* a pi 4 is probably a good option if you want to use the desktop

3) never got here
4) never got here
5) never got here


##### Terminal Install
Try that again  without the desktop (headless). This worked good:

1) Install Ubuntu MATE on TurtleBot PC 
* download Mate 18.04 64bit image for pi 3B + from Ubuntu Mate website (https://ubuntu-mate.org/download/arm64/)
* use pi-imager to load the image to SD card (https://www.raspberrypi.org/downloads/)
* insert SD card and boot pi
* use GUI to setup user account for Mate and keyboard setting yada yada
* do not login into desktop, instead press 'ctrl+alt+f1' to open a terminal
* login as your user, no need to login as root

* update the repository list - no need to upgrade

  `sudo apt update`

* install SSH for remote connection on remote computer - this is already installed on he Mate image but NOT on the Ubuntu install.  This makes no sense to me, but it will not hurt to run this command on both machines.  

 `sudo apt install openssh-server`

* test connectivity - both computers should be on the same network, prefferably plugged into ethernet cables and not using wifi

 `ip a`

  check that you get a valid ip address - take a picture of the terminal!

  try to ping the pi from a remote computer (on the network)

  `ping 192.168.xxx.yy`

  you should get byte transferred as shown below

  ```PING 192.168.254.22 (192.168.254.22) 56(84) bytes of data.
  64 bytes from 192.168.254.22: icmp_seq=1 ttl=64 time=0.599 ms
  64 bytes from 192.168.254.22: icmp_seq=2 ttl=64 time=0.620 ms
  64 bytes from 192.168.254.22: icmp_seq=3 ttl=64 time=0.624 ms
  ```
  next try to ssh in from the remote computer. Make sure openssh-server is on both machines

 `ssh <pi user>@<pi ip>`

  ssh was not working, so... I made some changes to /etc/ssh/sshd_config THIS WAS NOT THE FIX
  the fix is much easier, just run this on the host (the pi) and you should be good to go

  `sudo dpkg-reconfigure openssh-server`

  finally test that you can connect to the pi from the control computer
  notice how it shows that you have changed computers

  `thill@T1600-brwn305:~$ ssh thill@192.168.254.22`

  ```  thill@192.168.254.22's password:
  Welcome to Ubuntu 18.04.2 LTS (GNU/Linux 4.15.0-1032-raspi2 aarch64)

   * Documentation:  https://help.ubuntu.com
   * Management:     https://landscape.canonical.com
   * Support:        https://ubuntu.com/advantage


  0 packages can be updated.
  0 updates are security updates.

  New release '20.04.1 LTS' available.
  Run 'do-release-upgrade' to upgrade to it.

  Last login: Sat Oct 24 01:09:38 2020 from 192.168.254.45
  thill@mate18-pi3bp:~$
  ```
  this means that you are in, woop woop!

  now would be a good time to make a backup image... lol
  
  2) Install ROS on TurtleBot PC - Robotis Emanual calls the 'manual', do not use their script without invesitigating first
  install ROS following  the steps here http://wiki.ros.org/melodic/Installation/Ubuntu but do them through ssh
  
  thill@T1600-brwn305:~$ ssh <pi user>@<pi ip>

  then setup a workspace for ros called 'pi_ros'

  ``` ~$ mkdir -p ~/pi_ros/src
  ~$ cd pi_ros/
  ~/pi_ros$ catkin_make
  ~/$ echo "~/pi_ros/devel/setup.bash" >> ~/.bashrc
  source ~/pi_ros/devel/setup.bash 
  ```

  your workspace should compile without errors

* now install the turtlebot3 packeges following this https://emanual.robotis.com/docs/en/platform/turtlebot3/raspberry_pi_3_setup/ (SBC setup)

 

   3) Install Dependent Packages on TurtleBot PC - replace all instances of 'kinetic' with 'melodic'


   download the drivers from github, make sure you are in `~/pi_ros/src`  before you clone the repo
   
   ```
   $ cd ~/pi_ros/src
   $ git clone https://github.com/ROBOTIS-GIT/hls_lfcd_lds_driver.git
   $ git clone https://github.com/ROBOTIS-GIT/turtlebot3_msgs.git
   $ git clone https://github.com/ROBOTIS-GIT/turtlebot3.git
   ```

   go into turtlbot3 and delete some stuff (I am not sure why I am just following)

   $ cd ~/catkin_ws/src/turtlebot3
   $ rm -r turtlebot3_description/ turtlebot3_teleop/ turtlebot3_navigation/ turtlebot3_slam/ turtlebot3_example/
   
   install some Packages (changed kinetic to melodic)

   `$ sudo apt install ros-melodic-rosserial-python ros-melodic-tf`


   backout to the top of the workspace and build with catkin_make (what is -j1 ?)

   `$ cd ~/catkin_ws && catkin_make -j1`

   Everything seemed to work just fine - good news !

   TO DO  step 4,5 next

   4) USB Settings

   5) Network Configuration
