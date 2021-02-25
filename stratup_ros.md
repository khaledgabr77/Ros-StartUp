# Run ROS On Startup

 Imagin you developed ROS application, which can be launched with a single launch file.
But, There is still a problem. Actually you have to manually launch this file with a command line tool (roslaunch), and you wonder how you could make your application automatically start when you boot your computer, NUC, RaspberryPi!.
There are many ways to do this in linux including the systemd, rc.local file.

## 1- Systemd

### systemd setup

###### You need to create 3 files to setup systemd.

First file will start roscore, you need to make sure to fill in your [user name] where is says [TODO] and change melodic to whichever ROS version you are using.

```bash
sudo nano /etc/systemd/system/roscore.service
```



```bash
[Unit]
After=NetworkManager.service time-sync.target
[Service]
Type=forking
User=[TODO enter user name here]
ExecStart=/bin/bash -c ". /opt/ros/melodic/setup.bash; . /etc/ros/env.sh; roscore & while ! echo exit | nc localhost 11311 > /dev/null; do sleep 1; done"
[Install]
WantedBy=multi-user.target

```

This file stores your environment variables.

```bash
sudo nano /etc/ros/env.sh
```

```bash
#!/bin/sh
export ROS_HOSTNAME=$(hostname).local
export ROS_MASTER_URI=http://$ROS_HOSTNAME:11311
```

Second file defines a systemd service that runs your launch file. make sure to replace the [TODO] with your [user name].

```bash
sudo nano /etc/systemd/system/roslaunch.service
```

```bash
[Unit]
Requires=roscore.service
PartOf=roscore.service
After=NetworkManager.service time-sync.target roscore.service
[Service]
Type=simple
User=[TODO enter user name here]
ExecStart=/usr/sbin/roslaunch
[Install]
WantedBy=multi-user.target
```

Third file is responsible for running the launch file you specify. Make sure to enter your username and the name of your launch file

```bash
sudo nano /usr/sbin/roslaunch
```

```bash
#!/bin/bash
source ~/[your Workspace name]/devel/setup.bash
source /etc/ros/env.sh
export ROS_HOME=$(echo ~[TODO enter your username here)/.ros
[TODO place your roslaunch command here] &
# roslaunch at1_navigation at1.launch &
PID=$!
wait "$PID"
```

Next  execute and enable your systemd scripts by exceuting the following in a terminal:

```bash
sudo systemctl enable roscore.service
```

```bash

sudo systemctl enable roslaunch.service
```

```bash

sudo chmod +x /usr/sbin/roslaunch
```

You can reboot system now, Once the laptop is open again you can run. 

```bash
 reboot
 ```

```bash
 rostopic list
 ```