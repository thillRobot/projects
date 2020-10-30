## TWH logs - CARLA - open source vehicle simulator
## October 07, 2020 - October 14, 2020 - October 29, 2020

### Hardware

#### Test Server Computer:

* Computer: Dell t1600
* CPU:      Xeon CPU E31245 @ 3.30GHz × 8
* Graphics: Geforce GT 630 - > Geforce GTX 1650
* RAM:      8GB - > 16 GB
* OS:       Ubuntu 20.04/Ubuntu18.04

#### Test Client Computer:

* Computer: Intel NUC7i7BNH
* Graphics: KabyLake GT3e
* RAM:      16GB
* OS:       Ubuntu 18.04


### Required Software

* CARLA - the core of this project

* docker CE

* nvidia-docker2 (this requires nvidia drivers and driver version limits carla version)

* ROS

* ROS_BRIDGE

* NUMPY and PYGAME



### Software Installations

I installed 'docker CE' and 'nvidia-docker2' following the instructions that I was lead to from the carla docs
* https://carla.readthedocs.io/en/latest/build_docker/
* https://carla.readthedocs.io/en/latest/build_docker/#docker-ce Be careful not to install docker CE with apt and the script!
* https://carla.readthedocs.io/en/latest/build_docker/#nvidia-docker2


then I pulled a older vesion of carla 0.8.4. , this does not need to be repeated unless I change version

`sudo docker pull carlasim/carla:0.8.4`

then I pulled the two more recent version, I as trying to get around the library issues i was having

`sudo docker pull carlasim/carla:0.9.10`

this one in only on the 18.04 'sandbox computer' at the moment

`sudo docker pull carlasim/carla:0.9.10.1 `

so now we have three!

I installed numpy and pygame to run the client on the host


#### Software test

### CARLA Version 0.8.4 - Nearly Stable  (Stable is 0.8.2)

#### Start the CARLA server in a container

run the default script 'CarlaUE4.sh' in a carla 0.8.4 container and give a name 'carla'

`sudo docker run --name carlaserver -p 2000-2002:2000-2002 --runtime=nvidia --gpus all carlasim/carla:0.8.4 \
/bin/bash CarlaUE4.sh -quality-level=low -carla-server -benchmark -fps=10`

this start the server, now it is waiting for a client to connect

#### start a client on the host machine - NOT WORKING - PORT2000 CLOSED

change to the PythonClient directory and run one of the example scripts

`cd ~/carla_simulator/carla_084/PythonClient`
`./manual_control.py --autopilot`


#### start a client on the remote computer - This works

change to the PythonClient directory and run one of the example scripts

`cd ~/carla_simulator/carla_084/PythonClient`

`./manual_control.py --autopilot --host 192.168.x.x`

This open a PYGAME window and you can drive the car around with the keyboard.
You have to take the parking break off first!

At the moment the server<--->client relationship works!
The response is slow. I am only getting about 5 FPS on the remote NUC.



#### Shutdown the CARLA server (version 084 'ctrl-c' does not stop preocess)

You cannot just close the process, you have to stop the container
This process needs to be improved. I am supposed to be able to use the --name option to give
container a persistent name but I have not made this work yet.

For now, to stop the container you must first list the running containers

`sudo docker ps`

OR

`sudo docker container ls`

and you will get something like this:

```
CONTAINER ID        IMAGE                  COMMAND                  CREATED             STATUS              PORTS                              NAMES
7b2ab18618af        carlasim/carla:0.8.4   "/bin/bash CarlaUE4.…"   20 minutes ago      Up 20 minutes       0.0.0.0:2000-2002->2000-2002/tcp   elastic_kare
```


read the goofy name over on the right and that is the 'name' you will use to stop the containers

`sudo docker stop elastic_kare`



OK I have made some more progress

Run the server - notice I have added the opengl flag

`sudo docker run -p 2000-2002:2000-2002 --runtime=nvidia --gpus all carlasim/carla:0.8.4 /bin/bash CarlaUE4.sh DISPLAY= ./CarlaUE4.sh -opengl -carla-serve`r

Run the client - notice this is my script that I modified. Cool

`cd ~/carla_simlulator/carla_v084/PythonClient`

`/.manual_control_twh.py --autopilot --host 192.168.1.2 -q Low`



### CARLA Version 0.9.10 - Development Version

This requires the more modern nividia drivers, I installed  nvidia450

`sudo docker pull carlasim/carla:0.9.10`

I ran into this  errors: sh: 1: xdg-user-dir: not found

but i found a solution here (https://github.com/carla-simulator/carla/issues/3156)

There are still some warnings but it seems like the simulation has started.
-p allows one to one mapping of ports host - container

#### CARLA Server - The server is the world simulation

##### run the server in a container
This will run the script CarlaUE4.sh in the carla container. This starts the server under a random funny name.

`sudo docker run -e SDL_VIDEODRIVER=x11 -e DISPLAY=$DISPLAY -v /tmp/.X11-unix:/tmp/.X11-unix -p 2000-2002:2000-2002 -it --gpus all carlasim/carla:0.9.10 ./CarlaUE4.sh -opengl`

Using the --name option to choose a name is very useful. See below.

`sudo docker run --name carlaserver -e SDL_VIDEODRIVER=x11 -e DISPLAY=$DISPLAY -v /tmp/.X11-unix:/tmp/.X11-unix -p 2000-2002:2000-2002 -it --gpus all carlasim/carla:0.9.10 ./CarlaUE4.sh -opengl`

This will run BASH in the carla container without starting the simulator.

`sudo -E docker run --name carlaserver --privileged --rm --gpus all -it --net=host -e DISPLAY=$DISPLAY -v /tmp/.X11-unix:/tmp/.X11-unix:rw -it carlasim/carla:0.9.10 bash`

Do not run the docker as --privileged even though some forums may suggest it. This gives container root access to host, and this is very dangerous.
Someone suggested this. DO NOT DO IT

`sudo -E docker run --name carla --rm --gpus all -it --net=host -e DISPLAY=$DISPLAY -v /tmp/.X11-unix:/tmp/.X11-unix:rw carlasim/carla:0.9.10 /bin/bash ./CarlaUE4.sh -vulkan -benchmark -fps=25`

in version 0.9.10 you can ctrl-c to close the server, but I want to check that this is ok, i think the container is removed so it should be fine
unless you want to run that same container again with docker start or restart

##### runnning the server without sudo (root access)

If you are a regular user without without access to sudo, you must be added to the docker group. This does not require sudo because <user> owns $XAUTHORITY 

`sudo groupadd docker`

`sudo usermod -aG docker $USER`

Also, for the X11 stuff to work the container needs access to $XAUTHORITY , this is a common issue with GUI in containers 

`chmod 644 $XAUTHORITY`

Now you should be able to run the server with the `docker run` below. 

`docker run -e SDL_VIDEODRIVER=x11 -e DISPLAY=$DISPLAY -e XAUTHORITY=$XAUTHORITY -v /tmp/.X11-unix:/tmp/.X11-unix -v $XAUTHORITY:$XAUTHORITY -it --gpus all -p 2000-2002:2000-2002 carlasim/carla:0.9.10.1 ./CarlaUE4.sh -opengl`


#### CARLA Client - The client is a car driving in the world

##### run the client in ~/

Download and extract the appropriate version from Github. Here we are using: carla 0.9.10 (https://github.com/carla-simulator/carla)

I do not like this option because it seems like it should be available in the container...

On the client side I have had some trouble with the 'no module named carla issue' - https://github.com/carla-simulator/carla/issues/1137
this is related to properly installing the 'carla' python module from /carla/PythonAPI, i have it working, except not in a docker..
because that is not working... yet .... boo hoo, I am going to try something else ..
try to run the client on the local machine - not in container for now. - this seems to work pretty good - still would like both to be in a container
I bet it is just the paths...yep

In Ubuntu 20.04 (server machine) I downloaded and extracted carla 0.9.10 - 'pip3 install pygame' did not work so I had to use 'apt install python3-pygame'
i had to set the PYTHONPATH for the carla module to work. Basically the PYTHONPATH must include the path to .egg file for the right version of carla, I think that this is the same problem I am having in the docker container 'no module named carla'

Either way, Jared said "but generally we recommed y'all using Miniconda/Anaconda to create your own python 2 & 3 environments without any need for admin."

The client requires NUMPY and PYGAME

`pip3 install numpy pygame`

Set the PYTHONPATH (CARLA_ROOT is just intermediate variable to save length)

`export CARLA_ROOT=~/carla_simulator/carla_0910`

`export PYTHONPATH=$PYTHONPATH:${CARLA_ROOT}/PythonAPI/carla/dist/carla-0.9.10-py3.7-linux-x86_64.egg:${CARLA_ROOT}/PythonAPI/carla/agents:${CARLA_ROOT}/PythonAPI/carla`

Then, run the client script manual_control.py

`python3 ${CARLA_ROOT}/PythonAPI/examples/manual_control.py`

it works pretty good but...


sometimes this throws an error like this:
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




#### altemnatively run the client in the container - this is what I really want - This does not work yet

I would really like for the client and server to be in the docker container. To me it seems to make sense for me to be able to run both in the container. 

### Try to `run` the carla server in a docker container, and then `run` the client in the same container. 

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
### Alternatively, `run` the carla server in a docker container, and `exec` the the client in the same container as the server - not working 

`sudo docker exec -e PYTHONPATH=/home/carla/PythonAPI/carla/dist/carla-0.9.10-py3.7-linux-x86_64.egg:/home/carla/PythonAPI/carla/agents:/home/carla/PythonAPI/carla carla python3 PythonAPI/examples/manual_control.py`

`Traceback (most recent call last):
  File "PythonAPI/examples/manual_control.py", line 77, in <module>
    import carla
  File "/home/carla/PythonAPI/carla/dist/carla-0.9.10-py3.7-linux-x86_64.egg/carla/__init__.py", line 8, in <module>      YOU CAN SEE THE PATHS ARE WRONG HERE
  File "/home/carla/PythonAPI/carla/dist/carla-0.9.10-py3.7-linux-x86_64.egg/carla/libcarla.py", line 9, in <module>      THERE SHOULD NOT BE TWO carla directories
  File "/home/carla/PythonAPI/carla/dist/carla-0.9.10-py3.7-linux-x86_64.egg/carla/libcarla.py", line 7, in __bootstrap__ Actually you are wrong carla is the name of the container and the python module
ImportError: libxerces-c-3.2.so: cannot open shared object file: No such file or directory
`
### Trying something else: `run` the carla server in a docker container, and then `run` the client in a separate container. 
To do this use the --name option to force the containers to be different.

Start the carla server in a docker container named `carlaserver`

`sudo docker run --name carlaserver -e SDL_VIDEODRIVER=x11 -e DISPLAY=$DISPLAY -v /tmp/.X11-unix:/tmp/.X11-unix -p 2000-2002:2000-2002 -it --gpus all carlasim/carla:0.9.10 ./CarlaUE4.sh -opengl`

Then, start the the client in the different container named `carlaclient`

`sudo docker run --name carlaclient -e SDL_VIDEODRIVER=x11 -e DISPLAY=$DISPLAY -e PYTHONPATH=/home/carla/PythonAPI/carla/dist/carla-0.9.10-py3.7-linux-x86_64.egg:/home/carla/PythonAPI/carla/agents:/home/carla/PythonAPI/carla -v /tmp/.X11-unix:/tmp/.X11-unix -it --gpus all carlasim/carla:0.9.10 python3 PythonAPI/examples/manual_control.py`

This is not seem to be any different. 

### Mike was trying to help me fix the missing library file (libxerces-c-3.2.so) issue. This library issue was fixed by installing missinhg librbaries in container. See below. 

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

### Now we are going to repeat those tests with carla 0.9.10.1

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

### I was still stuck so I finally reached out to the guys at CARLA

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

`apt-get install vim` You probably do not need a text editor.


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



### THINGS TO DO:

* install and test ROS_BRIDGE
* figure out controls for first demo - drive a car        - Pretty Close
* clean up this document, it is a huge mess - A little better

* test client on local machine - not in docker - Done - Working
* test client on remote machine - not in docker - Done - Working
* test client on local machine - in a docker   - DONE - not working
* test server on local machine - in a docker - DONE - Working
* test server on local machine - not in a docker - NOT DONE

* try the stable version (0.8.4 or 0.8.2) - no ROS_BRIDGE
* turn on ROS_BRIDGE


