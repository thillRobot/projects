## TWH logs - CARLA - open source vehicle simulator
October 07, 2020 - October 14, 2020 - October 29, 2020

### Goals:

- [x] install and test ROS_BRIDGE in ros melodic
- [ ] install and test ROS_BRIDGE in ros noetic
- [ ] figure out steering wheel and pedal controls for demo - testdrive an autonomous car - Jon H. worked on this 
- [ ] record a *metric* for Dr. Canfield
- [ ] use RVIZ to visualize the data from carla

- [x] learn to use CONDA for python client - this will help with testing - done - big improvement
- [x] start the server in a different town, HDMaps/Town02.pcd  
- [x] change the town from the client side, HDMaps/Town02.pcd 
- [ ] document and test basic use of CARLA - not in a docker container

- [x] try the stable version (0.8.4 or 0.8.2) - does not have ROS_BRIDGE support I do not think
- [x] test client on local machine - not in docker - Done - Working
- [x] test client on remote machine - not in docker - Done - Working
- [x] test client on local machine - in a docker   - DONE - not working
- [x] test server on local machine - in a docker - DONE - Working
- [ ] test server on local machine - not in a docker - NOT DONE
- [ ] clean up this document, it is a huge mess - it is a little better

### Useful Resources

https://carla.readthedocs.io/en/latest/

I am not the first one doing this... what a surprise
https://usermanual.wiki/Document/CARLASetupGuideUbuntu.271743992/help  


### Hardware

#### Test Server Computer:

* Computer: Dell t1600
* CPU:      Xeon CPU E31245 @ 3.30GHz × 8
* Graphics: Geforce GT 630 - > Geforce GTX 1650
* RAM:      8GB - > 16 GB
* OS:       Ubuntu 20.04/Ubuntu18.04

#### Test Client Computer:

* Computer: Intel NUC7i7BNH
* CPU:      Intel i7
* Graphics: KabyLake GT3e
* RAM:      16GB
* OS:       Ubuntu 18.04


### Installing CARLA


#### Required/Related Software


* CARLA - the core of this project
* Python - Python3 is reccomended but apparently the client runs on Python2.
           This needs to be addressed.
* NUMPY and PYGAME
* docker CE
* nvidia-docker2 (this requires nvidia drivers and driver version limits carla version)
* ROS
* ROS_BRIDGE

#### Different Options for Installing CARLA
There are multple ways to install and run the CARLA package. Which is the right way, who knows.

1. Download and Extract from CARLA package from Github (https://github.com/carla-simulator/carla/releases) - if you just need a client - or do developement
  * CARLA Client - This is easy - requires numpy and pygame only 
  * CARLA Server -  This appears to be a very involved - requires building CARLA and UNREAL4 (https://carla.readthedocs.io/en/latest/build_linux/)
   
2. Install CARLA package with `apt install` - this should be very straight forward - but hard to change versions - for a general user
  * CARLA Client - this needs testing - this should be easy 
  * CARLA Server - this needs testing - it may be a pain to update versions - maybe not

3. Use Docker to pull and run a CARLA image (https://carla.readthedocs.io/en/latest/build_docker/) - for development and test - portability
  * CARLA Client - This should be easy, but this does not work - see bottom of this document
  * CARLA Server - This works good, but it did require some figuring out - see middle of this document

I am pursuing the Docker Approach for the server for several reasons. Mainly flexibility in testing. Currently, my working demo is a hybrid of approach 1 and 3 from above. It would nice if we had full functionality in a Docker container (method 3 only) because this would allow for complete portability. This may have to wait. Option 2 is a good idea also!

#### install docker 
I installed 'docker CE' and 'nvidia-docker2' following the instructions that I was lead to from the carla docs
* https://carla.readthedocs.io/en/latest/build_docker/
* https://carla.readthedocs.io/en/latest/build_docker/#docker-ce Be careful not to install docker CE with apt and the script!
* https://carla.readthedocs.io/en/latest/build_docker/#nvidia-docker2

##### using docker without sudo (root access)

If you are a regular user without without access to sudo, you must be added to the docker group. More details are dicsussed in the *post-installation steps*  here (https://docs.docker.com/engine/install/linux-postinstall/).

`sudo groupadd docker`

`sudo usermod -aG docker $USER`

Also run the following command to activate the changes to groups:

`newgrp docker`

Also, to run CARLA server with docker (the X11 port stuff), the container needs access to $XAUTHORITY , this is a common issue with GUI in containers. This does not require sudo because **user** owns $XAUTHORITY 

`chmod 644 $XAUTHORITY`

Now you should be able to run the `docker` commands below. All remaining `docker` commands should not require sudo.

#### pull CARLA images with docker
then I pulled a older vesion of carla 0.8.4. , this does not need to be repeated unless I change version

`docker pull carlasim/carla:0.8.4`

then I pulled the two more recent version, I as trying to get around the library issues i was having

`docker pull carlasim/carla:0.9.10`

This one in only on the 18.04 'sandbox computer' at the moment

`docker pull carlasim/carla:0.9.10.1 `

Now we have three images to choose from. 

#### Download and extract CARLA pre-compiled package from Github (reccomended option by CARLA for choosing versions)

Download and extract the appropriate version from Github. (https://github.com/carla-simulator/carla)
I am currently putting the package in ` ~/carla_simulator/carla_<version number> `
Here we show versions: 
 
 * carla 0.8.4
 * carla 0.9.10 
 * carla 0.9.10.1 

### Using CARLA Version 0.8.4
this is a hybrid of approach 1 (download and extract) and method 3 (run in docker) from the list above 

#### Start the CARLA server in a CARLA Version 0.8.4 container

run the default script 'CarlaUE4.sh' in a carla 0.8.4 container and give a name 'carlaserver'

`docker run --name carlaserver -p 2000-2002:2000-2002 --runtime=nvidia --gpus all carlasim/carla:0.8.4 \
/bin/bash CarlaUE4.sh -quality-level=low -carla-server -benchmark -fps=10`

this starts the server, now it is waiting for a client to connect

#### Start a client (0.8.4) on the host machine - NOT WORKING - PORT2000 CLOSED

change to the PythonClient directory and run one of the example scripts

`cd ~/carla_simulator/carla_084/PythonClient`
`./manual_control.py --autopilot`

#### Start a client (0.8.4) on a remote computer - This works

First change to the PythonClient directory and run one of the example scripts

`cd ~/carla_simulator/carla_084/PythonClient`

`./manual_control.py --autopilot --host 192.168.x.x`

#### Drive the car around with the keyboard in PYGAME.
You have to take the parking break off first! At the moment the server<--->client relationship works!

Note: the directory `/PythonClient` has changed `/PythonAPI` in the later versions 

#### Shutdown the CARLA server 

In version 084 'ctrl-c' does not stop the preocess

You cannot just close the process, you have to stop the container
This process needs to be improved. I am supposed to be able to use the --name option to give
container a persistent name but I have not made this work yet.

For now, to stop the container you must first list the running containers

`docker ps`

OR

`docker container ls`

and you will get something like this:

```
CONTAINER ID        IMAGE                  COMMAND                  CREATED             STATUS              PORTS                              NAMES
7b2ab18618af        carlasim/carla:0.8.4   "/bin/bash CarlaUE4.…"   20 minutes ago      Up 20 minutes       0.0.0.0:2000-2002->2000-2002/tcp   elastic_kare
```
read the goofy name over on the right and that is the 'name' you will use to stop the containers. This goofy name issue is solved with the `--name` option.

`docker stop elastic_kare`

OK I have made some more progress

Run the server - notice I have added the opengl flag

`docker run -p 2000-2002:2000-2002 --runtime=nvidia --gpus all carlasim/carla:0.8.4 /bin/bash CarlaUE4.sh DISPLAY= ./CarlaUE4.sh -opengl -carla-serve`r

Run the client - notice this is my script that I modified. Cool

`cd ~/carla_simlulator/carla_v084/PythonClient`

`/.manual_control_twh.py --autopilot --host 192.168.1.2 -q Low`

### Using CARLA Version 0.9.10 or 0.9.10.1 - Current Development Version

This requires the more modern nividia drivers, I installed  nvidia450 -> nvidia455

`docker pull carlasim/carla:0.9.10.1`

I ran into this  errors: sh: 1: xdg-user-dir: not found

but i found a solution here (https://github.com/carla-simulator/carla/issues/3156)

There are still some warnings but it seems like the simulation has started.
-p allows one to one mapping of ports host - container

#### CARLA Server - The server is the world simulation

##### run the server in a container
This will run the script CarlaUE4.sh in the carla container. This starts the server under a random funny name.

`sudo docker run -e SDL_VIDEODRIVER=x11 -e DISPLAY=$DISPLAY -v /tmp/.X11-unix:/tmp/.X11-unix -p 2000-2002:2000-2002 -it --gpus all carlasim/carla:0.9.10.1 ./CarlaUE4.sh -opengl`

Using the --name option to choose a name is very useful. See below.

`sudo docker run --name carlaserver -e SDL_VIDEODRIVER=x11 -e DISPLAY=$DISPLAY -v /tmp/.X11-unix:/tmp/.X11-unix -p 2000-2002:2000-2002 -it --gpus all carlasim/carla:0.9.10.1 ./CarlaUE4.sh -opengl -benchmark fps=20`

If you are not using sudo (preferred) then use the following line. 

`docker run --name carlaserver -e SDL_VIDEODRIVER=x11 -e DISPLAY=$DISPLAY -e XAUTHORITY=$XAUTHORITY -v /tmp/.X11-unix:/tmp/.X11-unix -v $XAUTHORITY:$XAUTHORITY -it --gpus all -p 2000-2002:2000-2002 carlasim/carla:0.9.10.1 ./CarlaUE4.sh -opengl`

This will run BASH in the carla container without starting the simulator.

`sudo docker run --name carlaserver --rm --gpus all -it --net=host -e DISPLAY=$DISPLAY -v /tmp/.X11-unix:/tmp/.X11-unix:rw -it carlasim/carla:0.9.10.1 bash`

Do not run the docker as --privileged even though some forums may suggest it. This gives container root access to host, and this is very dangerous.
Someone suggested this in a forum somewhere, but that does not mean it is a good idea.

In version 0.9.10 you can ctrl-c to close the server, but I want to check that this is ok, i think the container is removed so it should be fine
unless you want to run that same container again with docker start or restart

For now, you have to remove the container before you can start it again. This should be fixed but it is not at the moment.

`docker container rm carlaserver`

#### CARLA Client - The client is a car driving in the world server

I do not like this option because it seems like it should be available in the container...
On the client side I have had some trouble with the 'no module named carla issue' - https://github.com/carla-simulator/carla/issues/1137
this is related to properly setting the path for the 'carla' python module from /carla/PythonAPI. 

In Ubuntu 20.04 (server machine) I downloaded and extracted carla 0.9.10 - 'pip3 install pygame' did not work so I had to use 'apt install python3-pygame'
i had to set the PYTHONPATH for the carla module to work. Basically the PYTHONPATH must include the path to .egg file for the right version of carla, I think that this is the same problem I am having in the docker container 'no module named carla'

##### Running the client in users home directory (~/) of the local (server) machine 
The client requires NUMPY and PYGAME (https://carla.readthedocs.io/en/latest/start_quickstart/). 
Do I need the `--user` option ? What does that even do? I think I know.

Before you can run the client (0.9.10.10) you have to set PYTHONPATH to .egg file. (CARLA_ROOT is just intermediate variable to save length)

`export CARLA_ROOT=~/carla_simulator/carla_09101`

`export PYTHONPATH=$PYTHONPATH:${CARLA_ROOT}/PythonAPI/carla/dist/carla-0.9.10-py3.7-linux-x86_64.egg:${CARLA_ROOT}/PythonAPI/carla/agents:${CARLA_ROOT}/PythonAPI/carla`

OR if you are using Python2.7 use the appropriate .egg file

`export PYTHONPATH=$PYTHONPATH:${CARLA_ROOT}/PythonAPI/carla/dist/carla-0.9.10-py2.7-linux-x86_64.egg:${CARLA_ROOT}/PythonAPI/carla/agents:${CARLA_ROOT}/PythonAPI/carla`

Then, you can run *some* of the examples in `/PythonAPI/examples` and `/PythonAPI/utils`, but several of the scripts fail.

##### Run the Client in CONDA Environment - This saves time and is preferred method during testing
Either way, Jared said *but generally we recommed y'all using Miniconda/Anaconda to create your own python 2 & 3 environments without any need for admin.*
So, I finally tested it with CONDA . Install conda following instructions here (https://docs.anaconda.com/anaconda/install/linux/). Use CONDA for a virtual environment I have setup for conveinence. This way you do not have to set the paths each time.

Create a environment to use the client in (this only needs to be done once)
this environment will have python3.7 installed

`conda create --name carla09101 python=3.7`

then actitvate the environment (this needs to be done at the start of each session)

`conda activate carla09101`

finally add the paths to the conda environment so that you do not have to do this each time
this line shows to set a env var permanently in the conda environment

`conda env config vars set CARLA_ROOT=~/carla_simulator/carla_09101`

re-actitvate the environment after setting vars 

`conda activate carla09101`

`conda env config vars set PYTHONPATH=$PYTHONPATH:${CARLA_ROOT}/PythonAPI/carla/dist/carla-0.9.10-py3.7-linux-x86_64.egg:${CARLA_ROOT}/PythonAPI/carla/agents:${CARLA_ROOT}/PythonAPI/carla`

re-actitvate the environment after setting vars again (could this be combined?)

`conda activate carla09101`

You need to have `numpy` and `pygame` installed. The CARLA website reccomends doing like this. 

`pip3 install --user numpy pygame`

However, you can also install them with pip and the `requirements.txt` file. I am not sure which is better. It seems that using the requirements.txt in conda takes long time. What does this mean, I dont know.

now that `CARLA_ROOT` is set you can install the python requirements with the folllowing:

`pip3 install -r ${CARLA_ROOT}/PythonAPI/examples/requirements.txt`

`automatic_control.py` requires the networkx module to be install - i used conda to install it (the env most still be active of course)

`conda install networkx`

`conda activate carla09101`

This starts a client and lets you drive with PYGAME. Also because these scripts at home they will easy to modify.

`python3 ${CARLA_ROOT}/PythonAPI/examples/manual_control.py`

If the client is remote then you have to inlcude the IP address of the host.

`python3 ${CARLA_ROOT}/PythonAPI/examples/manual_control.py --host 192.168.254.45` 

Sometimes is throws an error like this. Typically if you try again it will work...

```
'No recommended values for 'speed' attribute
Traceback (most recent call last):
  File "/home/thill/carla_simulator/carla_0910//PythonAPI/examples/manual_control.py", line 1137, in <module>
    main()
  File "/home/thill/carla_simulator/carla_0910//PythonAPI/examples/manual_control.py", line 1129, in main
    game_loop(args)
  File "/home/thill/carla_simulator/carla_0910//PythonAPI/examples/manual_control.py", line 1046, in game_loop
    controller = KeyboardControl(world, args.autopilot)
  File "/home/thill/carla_simulator/carla_0910//PythonAPI/examples/manual_control.py", line 292, in __init__
    world.player.set_autopilot(self._autopilot_enabled)
RuntimeError: time-out of 2000ms while waiting for the simulator, make sure the simulator is ready and connected to 127.0.0.1:2000'
```
I think this may be another app using that port, but I am not sure.

##### Configuring the CARLA server from a client
A new useful feature I have just discovered is `/PythonAPI/utils/config.py`. This scripts is used to configure a running CARLA server. You can do things like change the town map and other things. This is very useful becuase is it a pain (so much that I was unable to do so!) from the server side. I guess this makes sensse...

Here is an example that shows how to change the town map.

`python3 ${CARLA_ROOT}/PythonAPI/util/config.py --map Town04`

And this line changes the weather. 

`python3 ${CARLA_ROOT}/PythonAPI/util/config.py --weather HardRainNoon`


#### CARLA ROS-BRIDGE - This gives us access to data from the simulation in ROS

Follow the instructions on the ROS-BRIDGE github (https://github.com/carla-simulator/ros-bridge)

Create a catkin workspace and install carla_ros_bridge package
```
#setup folder structure
mkdir -p ~/carla-ros-bridge/catkin_ws/src
cd ~/carla-ros-bridge
git clone https://github.com/carla-simulator/ros-bridge.git
cd ros-bridge
git submodule update --init
cd ../catkin_ws/src
ln -s ../../ros-bridge
source /opt/ros/<kinetic or melodic or noetic>/setup.bash
cd ..

#install required ros-dependencies
rosdep update
rosdep install --from-paths src --ignore-src -r

#build
catkin_make
```

##### run the server

`docker run --name carlaserver -e SDL_VIDEODRIVER=x11 -e DISPLAY=$DISPLAY -e XAUTHORITY=$XAUTHORITY -v /tmp/.X11-unix:/tmp/.X11-unix -v $XAUTHORITY:$XAUTHORITY -it --gpus all -p 2000-2002:2000-2002 carlasim/carla:0.9.10.1 ./CarlaUE4.sh -opengl`

##### run the ROS BRIDGE (with client)

`export CARLA_ROOT=~/carla_simulator/carla_09101`

`export PYTHONPATH=$PYTHONPATH:${CARLA_ROOT}/PythonAPI/carla/dist/carla-0.9.10-py2.7-linux-x86_64.egg:${CARLA_ROOT}/PythonAPI/carla/agents:${CARLA_ROOT}/PythonAPI/carla`

`roslaunch carla_ros_bridge carla_ros_bridge_with_example_ego_vehicle.launch`

##### run rostopic to test

`rostopic list`

`rostopic echo /carla/ego_vehicle/imu/imu1`

You should now be able to see the data from the simulator in ROS, cool.



#### alternatively run the client in the container - this is what I really want - This does not work yet

I would really like for the client and server to be in the docker container. To me it seems to make sense for me to be able to run both in the container. 

##### Try to `run` the carla server in a docker container, and then `run` the client in the same container. 

Start the carla server in a docker container. Apparantely I was not using the name option. I need to add the name option here. 

`sudo docker run -e SDL_VIDEODRIVER=x11 -e DISPLAY=$DISPLAY -v /tmp/.X11-unix:/tmp/.X11-unix -p 2000-2002:2000-2002 -it --gpus all carlasim/carla:0.9.10 ./CarlaUE4.sh -opengl`

I have discovered that if you set the paths CORRECTLY then the python scripts run without import errors, but then there are other errors. Fixing these errors is my main goal in this section. I also realized that this seems to work just the same. Only the first python path is required (i am not sure if this is ok). 

`export CARLA_ROOT=~/`
`export PYTHONPATH=$PYTHONPATH:${CARLA_ROOT}/PythonAPI/carla/dist/carla-0.9.10-py3.7-linux-x86_64.egg:${CARLA_ROOT}/PythonAPI/carla/agents:${CARLA_ROOT}/PythonAPI/carla`

use to -e or --env to set env vars in the container

`--env PYTHONPATH=~/PythonAPI/carla/dist/carla-0.9.10-py3.7-linux-x86_64.egg:~/PythonAPI/carla/agents:~/PythonAPI/carla`

`-e PYTHONPATH=/home/carla/PythonAPI/carla/dist/carla-0.9.10-py3.7-linux-x86_64.egg`

`run` the carla server in a docker container

`sudo docker run --name carla -e SDL_VIDEODRIVER=x11 -e DISPLAY=$DISPLAY -v /tmp/.X11-unix:/tmp/.X11-unix -p 2000-2002:2000-2002 -it --gpus all carlasim/carla:0.9.10 ./CarlaUE4.sh -opengl`

Next, `run` a client on the server machine this time so there is no ip needed, you do need the --net=host option for the client and server to talk.

`sudo docker run -e SDL_VIDEODRIVER=x11 -e DISPLAY=$DISPLAY -v /tmp/.X11-unix:/tmp/.X11-unix --net=host -it --gpus all carlasim/carla:0.9.10 /PythonAPI/examples/manual_control.py --env CARLA_ROOT=~/ --env PYTHONPATH=~/PythonAPI/carla/dist/carla-0.9.10-py3.7-linux-x86_64.egg:~/PythonAPI/carla/agents:~/PythonAPI/carla`

, 
##### Alternatively,`run` the carla server in a docker container, and `exec` the the client in the same container as the server - not working 

`sudo docker exec -e PYTHONPATH=/home/carla/PythonAPI/carla/dist/carla-0.9.10-py3.7-linux-x86_64.egg:/home/carla/PythonAPI/carla/agents:/home/carla/PythonAPI/carla carla python3 PythonAPI/examples/manual_control.py`

`Traceback (most recent call last):
  File "PythonAPI/examples/manual_control.py", line 77, in <module>
    import carla
  File "/home/carla/PythonAPI/carla/dist/carla-0.9.10-py3.7-linux-x86_64.egg/carla/__init__.py", line 8, in <module>      YOU CAN SEE THE PATHS ARE WRONG HERE
  File "/home/carla/PythonAPI/carla/dist/carla-0.9.10-py3.7-linux-x86_64.egg/carla/libcarla.py", line 9, in <module>      THERE SHOULD NOT BE TWO carla directories
  File "/home/carla/PythonAPI/carla/dist/carla-0.9.10-py3.7-linux-x86_64.egg/carla/libcarla.py", line 7, in __bootstrap__ Actually you are wrong carla is the name of the container and the python module
ImportError: libxerces-c-3.2.so: cannot open shared object file: No such file or directory
`
##### Something else: `run` the carla server in a docker container, and then `run` the client in a separate container. 
To do this use the --name option to force the containers to be different.

Start the carla server in a docker container named `carlaserver`

`sudo docker run --name carlaserver -e SDL_VIDEODRIVER=x11 -e DISPLAY=$DISPLAY -v /tmp/.X11-unix:/tmp/.X11-unix -p 2000-2002:2000-2002 -it --gpus all carlasim/carla:0.9.10 ./CarlaUE4.sh -opengl`

Then, start the the client in the different container named `carlaclient`

`sudo docker run --name carlaclient -e SDL_VIDEODRIVER=x11 -e DISPLAY=$DISPLAY -e PYTHONPATH=/home/carla/PythonAPI/carla/dist/carla-0.9.10-py3.7-linux-x86_64.egg:/home/carla/PythonAPI/carla/agents:/home/carla/PythonAPI/carla -v /tmp/.X11-unix:/tmp/.X11-unix -it --gpus all carlasim/carla:0.9.10 python3 PythonAPI/examples/manual_control.py`

This is not seem to be any different. 

##### Mike was trying to help me fix the missing library file (libxerces-c-3.2.so) issue. This library issue was fixed by installing missinhg librbaries in container. See below. 

Start the server in a container, use -v to mount /usr/lib/x86_64-linux-gnu/:/usr/local/host/lib on the host computer

`sudo docker run --name carlaserver -e SDL_VIDEODRIVER=x11 -e DISPLAY=$DISPLAY -v /tmp/.X11-unix:/tmp/.X11-unix -v /usr/lib/x86_64-linux-gnu/:/usr/local/host/lib -p 2000-2002:2000-2002 -it --gpus all carlasim/carla:0.9.10 ./CarlaUE4.sh -opengl`

OR with the LD_LIBRARY_PATH variable set

`sudo docker run --name carlaserver -e SDL_VIDEODRIVER=x11 -e DISPLAY=$DISPLAY -v /tmp/.X11-unix:/tmp/.X11-unix -v /usr/lib/x86_64-linux-gnu/:/usr/local/host/lib -e LD_LIBRARY_PATH=/usr/lib/x86_64-linux-gnu:/usr/lib/i386-linux-gnu:/usr/local/nvidia/lib:/usr/local/nvidia/lib64:/usr/local/nvidia/lib:/usr/local/nvidia/lib64:/usr/local/host/lib -p 2000-2002:2000-2002 -it --gpus all carlasim/carla:0.9.10 ./CarlaUE4.sh -opengl`

Now try the client side. We are trying to mount the library in host

`sudo docker exec -e PYTHONPATH=/home/carla/PythonAPI/carla/dist/carla-0.9.10-py3.7-linux-x86_64.egg:/home/carla/PythonAPI/carla/agents:/home/carla/PythonAPI/carla carlaserver python3 PythonAPI/examples/manual_control.py
[sudo] password for thill:
Traceback (most recent call last):
  File "PythonAPI/examples/manual_control.py", line 77, in <module>
    import carla
  File "/home/carla/PythonAPI/carla/dist/carla-0.9.10-py3.7-linux-x86_64.egg/carla/__init__.py", line 8, in <module>
  File "/home/carla/PythonAPI/carla/dist/carla-0.9.10-py3.7-linux-x86_64.egg/carla/libcarla.py", line 9, in <module>
  File "/home/carla/PythonAPI/carla/dist/carla-0.9.10-py3.7-linux-x86_64.egg/carla/libcarla.py", line 7, in __bootstrap__
ImportError: libxerces-c-3.2.so: cannot open shared object file: No such file or directory`

That did not fix the libxerces issue, at least we tried. Thanks Mike.

OKIE DOKIE!  I think i figured out the libxerces-c issue. well, maybe not but I think someone else did. Look at version 0.9.10.1
(https://github.com/carla-simulator/carla/releases)

"Fixed dependency of library Xerces-c on package" - I think they might mean 'of'
Well here we go with the latest version (16 days old).

##### Now we are going to repeat those tests with carla 0.9.10.1

run bash in the container - works

`sudo -E docker run --name carla --privileged --rm --gpus all -it --net=host -e DISPLAY=$DISPLAY -v /tmp/.X11-unix:/tmp/.X11-unix:rw -it carlasim/carla:0.9.10.1 bash`

Start the carla server in a docker container

`sudo docker run --name carla -e SDL_VIDEODRIVER=x11 -e DISPLAY=$DISPLAY -v /tmp/.X11-unix:/tmp/.X11-unix -p 2000-2002:2000-2002 -it --gpus all carlasim/carla:0.9.10.1 ./CarlaUE4.sh -opengl`

Then, start the the client in the same container as the server - not working

`sudo docker exec -e PYTHONPATH=/home/carla/PythonAPI/carla/dist/carla-0.9.10-py3.7-linux-x86_64.egg:/home/carla/PythonAPI/carla/agents:/home/carla/PythonAPI/carla carla python3 PythonAPI/examples/manual_control.py`

`Traceback (most recent call last):
  File "PythonAPI/examples/manual_control.py", line 77, in <module>
    import carla
  File "/home/carla/PythonAPI/carla/dist/carla-0.9.10-py3.7-linux-x86_64.egg/carla/__init__.py", line 8, in <module>
  File "/home/carla/PythonAPI/carla/dist/carla-0.9.10-py3.7-linux-x86_64.egg/carla/libcarla.py", line 9, in <module>
  File "/home/carla/PythonAPI/carla/dist/carla-0.9.10-py3.7-linux-x86_64.egg/carla/libcarla.py", line 7, in __bootstrap__
ImportError: libjpeg.so.8: cannot open shared object file: No such file or directory`

##### I was still stuck so I finally reached out to the guys at CARLA

Wow, another similar error, different library. Does this mean that they fixed the xerces error and not this one? I am a little confused about that. At this point I have spent some time on this. I finally broke down and asked the CARLA team. I posted my question. I was terrified that they would make fun of me, but they were vey helpful and responsive. Here is my post (https://github.com/carla-simulator/carla/issues/3236#issuecomment-711022225):

*I am having a similar issue while trying to run the PythonAPI in a carlasim/carla:0.9.10.1 docker container. The server runs with some warnings, but the client does not run and shows the following error when I run the client inside the container.*

```
  File "PythonAPI/examples/manual_control.py", line 77, in <module>
    import carla
  File "/home/carla/PythonAPI/carla/dist/carla-0.9.10-py3.7-linux-x86_64.egg/carla/__init__.py", line 8, in <module>
  File "/home/carla/PythonAPI/carla/dist/carla-0.9.10-py3.7-linux-x86_64.egg/carla/libcarla.py", line 9, in <module>
  File "/home/carla/PythonAPI/carla/dist/carla-0.9.10-py3.7-linux-x86_64.egg/carla/libcarla.py", line 7, in __bootstrap__
ImportError: libjpeg.so.8: cannot open shared object file: No such file or directory

```

*Has anyone successfully run the PythonAPI inside a carla docker container? I have read #2909, and #3236, but I have not found a solution yet. I am new to docker and carla so it may be something simple that I am doing wrong.*

As you can read in the link above the CARLA team was veryh helpful, but their suggestions have only lead me to more issues. Nicoloas said that carla is not runtime for the client (I am not sure what he means exactly), and that I will have to go into the client with bash and install the libraries with `apt-get`. This is very counter intutive to me.

##### most recent attempt at running carla client in container - I am trying to following suggestions from carla team.

For any of this to work the docker must be able to access the internet, to do this edit the /etc/default/docker file and give DNS ip address
add this line to the bottom of /etc/default/docker to allow a container to access the internet

`DOCKER_OPTS="--dns <your_dns_server_1> --dns <your_dns_server_2>"`

You can use `echo` to so append the file from the coammand line. It is shown below.

`sudo echo "DOCKER_OPTS=" --dns 192.168.254.254" >> /etc/default/docker"`

Then restart docker.

`sudo service docker restart`

Now that docker can connect to the web, start bash in a running container

`sudo docker exec -it -u 0:0 <container_name> /bin/bash`

`apt-get update`

`apt-get install vim` You probably do not need a text editor, but it cannot hurt right?

`apt-get install python3-pip` This might be the problem. Nicholas said he does not use pip in the container. 

`pip3 install -r PythonAPI/examples/requirements.txt` I do not think running requirements.txt is neccesary

`apt-get install libjpeg-turbo8 libtiff-dev libxerces-c3.2` Notice the `libjpeg` is in `libjpeg-turbo`


now execute the client in the running container- throws weird error (document this!)

 `python3 PythonAPI/examples/manual_control.py`

instead, exit bash in the container and run it from the outside 

`exit`

`sudo docker exec -e PYTHONPATH=/home/carla/PythonAPI/carla/dist/carla-0.9.10-py3.7-linux-x86_64.egg carlaserver python3 PythonAPI/examples/manual_control.py`

OK, more news! Running the installs in the container get us past the library issue but, now we are running into another issue with the python client related to the fonts and other fun things. 

Nicholas from CARLA team said first test that you can run 'tutorial.py'. OK, lets try that. Also, he said that he uses apt-get and not pip. this makes sense in the docker. let try this one more time.

`sudo docker exec -e PYTHONPATH=/home/carla/PythonAPI/carla/dist/carla-0.9.10-py3.7-linux-x86_64.egg carlaserver python3 PythonAPI/examples/tutorial.py`


