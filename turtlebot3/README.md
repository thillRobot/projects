### this is a log for working with the rasp pi b+ for turtlebot3

#### I brought a pi3B+ home to test

here is what I have done so far:
* downloaded Mate 18.04 64bit image for pi 3B + from Ubuntu Mate website
* used pi-imager to load the image to SD card
* open MATE desktop and it seems to work, graphics are terribly slow
* i install ros successfully and tested roscore
* i did some other things then it crashed, this seems to be a common brought

I think that we should just stay headless and I predict that issue will go away, but we will see.

lets ttry that again:
* download Mate 18.04 64bit image for pi 3B + from Ubuntu Mate website
* used pi-imager to load the image to SD card
* use GUI to setup account and keyboard yada yada
* do not login into desktop, press: 'ctrl+alt+f1' to open a terminal
* login as your user, not root for now


* update the repository list - no need to upgrade

  `sudo apt update`

* install SSH for remote connection - this is already install on Mate image

  `sudo apt install openssh-server`

* test connectivity

  `ip a`

  check that you get a valid ip address - take a picture of the terminal!

  try to ping the pi from a remote computer (on the network)

  `ping 192.168.xxx.yy`

  you should get byte transferred as shown below

  `PING 192.168.254.22 (192.168.254.22) 56(84) bytes of data.
  64 bytes from 192.168.254.22: icmp_seq=1 ttl=64 time=0.599 ms
  64 bytes from 192.168.254.22: icmp_seq=2 ttl=64 time=0.620 ms
  64 bytes from 192.168.254.22: icmp_seq=3 ttl=64 time=0.624 ms`

  next try to ssh in from the remote computer. Make sure openssh-server is on both machines

  `ssh <user on pi>@<ip of pi>`

  ssh was not working, so... I made some changes to /etc/ssh/sshd_config THIS WAS NOT THE FIX
  the fix is much easier, just run this on the host (the pi) and you should be good to go

  `sudo dpkg-reconfigure openssh-server`

  finally test that you can connect to the pi from the control computer
  notice how it shows that you have changed computers

  `thill@T1600-brwn305:~$ ssh thill@192.168.254.22`

  `thill@192.168.254.22's password:
  Welcome to Ubuntu 18.04.2 LTS (GNU/Linux 4.15.0-1032-raspi2 aarch64)

   * Documentation:  https://help.ubuntu.com
   * Management:     https://landscape.canonical.com
   * Support:        https://ubuntu.com/advantage


  0 packages can be updated.
  0 updates are security updates.

  New release '20.04.1 LTS' available.
  Run 'do-release-upgrade' to upgrade to it.

  Last login: Sat Oct 24 01:09:38 2020 from 192.168.254.45
  thill@mate18-pi3bp:~$ `

  now would be a good time to make a backup image... lol

  lets install some packages instead, follow the steps here http://wiki.ros.org/melodic/Installation/Ubuntu but do them through ssh
