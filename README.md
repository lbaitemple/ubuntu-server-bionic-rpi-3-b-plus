# Ubuntu Server 18.04.2 (Bionic) ARM image for Raspberry Pi 3 B+
### Download image
https://github.com/AbelCS/ubuntu-server-bionic-rpi-3-b-plus/releases


## Instructions
### step 1: Upgrade System 
Burn the image to SD card with dd/etcher/DiskWritter or your favorite tool.

Username: **ubuntu**  
Password: **ubuntu**

Check your system architecture, you can use
```
 uname -m
```

SSH is enabled by default, so you can login directly after first boot. However, it is recommended to connect to ethernet and use a HDMI screen to upgrade the system first. 

```
sudo apt-get update
sudo apt-get upgrade -y
```

If any lock files presented, please sudo remove them and 
```
sudo rm /var/lib/dpkg/lock*
sudo dpkg --configure -a
sudo apt-get upgrade -y
```
#### upgrade may take up to 30 mins. When a selection is requested, please use TAB key to select yes option.

### step 2: Setup I2C
After the system is upgraded in step 1, you can ssh to the raspberry pi in step 2 when your computer and your pi are on the same LAN. 


After login, you can use the following commands by copying and pasting into the ssh shell
When everything is completed, you will need to do the following commands to enable I2C
```
sudo apt-get install python-pip python-pil  i2c-tools mosquitto-clients -y
sudo pip install Adafruit_SSD1306 RPi.GPIO
```

To enable I2c permission
```
sudo chgrp i2c /dev/i2c-1
sudo chmod 666 /dev/i2c-1
sudo usermod -G i2c $USER
```

If everything wents through successfully, please go to step 3.

If you do not see the file /dev/i2c-1. Please add i2c in configure file
```
sudo nano /boot/firmware/config.txt
```

Make sure you have (I2C, SPI and UART)
```
dtparam=i2c_arm=on
dtparam=spi=on
enable_uart=1
```
add the following line to the file /etc/modules.
```
i2c-dev
```
To enable I2c permission
```
sudo chgrp i2c /dev/i2c-1
sudo chmod 666 /dev/i2c-1
sudo usermod -G i2c $USER
```


### step 3: Setup everything for IP 

* To get wireless connection working on boot you must edit **/etc/netplan/01-rpi-3-network.yaml** present in *cloudimg-rootfs* partition in your sdcard and add your SSID and PASSWORD.
Open file
```
sudo nano /etc/netplan/01-rpi-3-network.yaml
```
Please include the following content and make sure you generate your password hash (see line include command 

 ![#f03c15](https://placehold.it/15/f03c15/000000?text=+) `echo -n [password] | iconv -t UTF-16LE | openssl md4`
 
```
network:
        version: 2
        renderer: networkd
        ethernets:
                eth0:
                  optional: true
                  dhcp4: true
        wifis:
             wlan0:
                optional: true
                dhcp4: true
                access-points:
                        "L5GLB":
                                password: "password123"
                        tusecurewireless:
                                auth:
                                   key-management: eap
                                   password: hash:[insert hash value]
                                   method: peap
                                   identity: lbai

```

To apply netplan
```
sudo netplan --debug generate
sudo netplan try

```
You may have to reboot to get IP address up. Now you can setup MQTT client in startup script file
```
cd ~
git clone https://github.com/lbaitemple/ubuntu-server-bionic-rpi-3-b-plus
cp ubuntu-server-bionic-rpi-3-b-plus/newtest2.sh ~/test2.sh
cp ubuntu-server-bionic-rpi-3-b-plus/stats.py ~/
chmod +x test2.sh
```

You can open test2.sh and modify cloud MQTT setting. If you do not have a cloud MQTT account, please go to https://www.cloudmqtt.com/ to setup one free account. 
```
nano ~/test2.sh
```

You will need to ensure a startup service to enable network
```
sudo systemctl is-enabled systemd-networkd-wait-online.service
sudo systemctl enable systemd-networkd-wait-online.service
```
Now, you will need to create a startup service
```
sudo cp ~/ubuntu-server-bionic-rpi-3-b-plus/ipaddress.service /lib/systemd/system
sudo systemctl daemon-reload
sudo systemctl enable  ipaddress
sudo systemctl start  ipaddress
```
You can setup a MQTT subscriber to wait the ip address is published to the MQTT topic. Also, you should be able to see the IP address on OLED screen if you connect your I2C OLED screen (https://esphome.io/components/display/ssd1306.html) to your Pi.

#### optional step 3a: display ip address without login (with a monitor)
open a file 

```
sudo nano /etc/issue
```
add two lines in the bottom of the file
```
eth0: \4{eth0}
wlan0: \4{wlan0}
```

### step 4: Increase swap memory
When you compile files, you may need a larger swap memory becasue raspberry pi 3 has only 1GB memory.

Enter the command as follows to setup 4G swap space
```
sudo dd if=/dev/zero of=/swap1 bs=1M count=4096
sudo mkswap /swap1
sudo swapon /swap1
```
take around 5 mins to create 4G swap space. If you need to include the swap space during every bootup, you can open a file
```
sudo nano /etc/fstab
```
Add a line to the bottom of the file
```
/swap1 swap swap
```
close and save the file. You can check if there is a swap memory by typing
```
free -th
```
### step 5: Install ROS
```
sudo sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'
sudo apt-key adv --keyserver hkp://ha.pool.sks-keyservers.net:80 --recv-key C1CF6E31E6BADE8868B172B4F42ED6FBAB17C654
sudo apt-get update
sudo apt-get install ros-melodic-desktop -y
sudo rosdep init
rosdep update
echo "source /opt/ros/melodic/setup.bash" >> ~/.bashrc
```

### step 6: Install ROS2  [make sure you have a 32-bit image installed]
https://index.ros.org/doc/ros2/Installation/Dashing/Linux-Development-Setup/


```
sudo locale-gen en_US en_US.UTF-8
sudo update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8
export LANG=en_US.UTF-8
sudo apt update && sudo apt install curl gnupg2 lsb-release
curl -s https://raw.githubusercontent.com/ros/rosdistro/master/ros.asc | sudo apt-key add -
sudo sh -c 'echo "deb [arch=armhf] http://packages.ros.org/ros2/ubuntu `lsb_release -cs` main" > /etc/apt/sources.list.d/ros2-latest.list'
sudo apt update
sudo apt install -y \
  build-essential \
  cmake \
  git \
  python3-colcon-common-extensions \
  python3-pip \
  python-rosdep \
  python3-vcstool \
  wget
python3 -m pip install -U \
  argcomplete \
  flake8 \
  flake8-blind-except \
  flake8-builtins \
  flake8-class-newline \
  flake8-comprehensions \
  flake8-deprecated \
  flake8-docstrings \
  flake8-import-order \
  flake8-quotes \
  pytest-repeat \
  pytest-rerunfailures \
  pytest \
  pytest-cov \
  pytest-runner \
  setuptools
sudo apt install --no-install-recommends -y \
  libasio-dev \
  libtinyxml2-dev
sudo apt install --no-install-recommends -y \
  libcunit1-dev  
```
Get ROS2 Code
```
mkdir -p ~/ros2_dashing/src
cd ~/ros2_dashing
wget https://raw.githubusercontent.com/ros2/ros2/dashing/ros2.repos
vcs import src < ros2.repos
```
Install dependencies using rosdep
```
sudo rosdep init
rosdep update
rosdep install --from-paths src --ignore-src --rosdistro dashing -y --skip-keys "console_bridge fastcdr fastrtps libopensplice67 libopensplice69 rti-connext-dds-5.3.1 urdfdom_headers"
```

Build the code in the workspace
```
cd ~/ros2_dashing/
colcon build --symlink-install
````
##### take really long 6-7 hours. Take a break and get a cup of coffee.

```
colcon build --symlink-install --merge-install
```

Try some examples
```
. ~/ros2_dashing/install/local_setup.bash
ros2 run demo_nodes_cpp talker
```
In another terminal source the setup file and then run a listener:
```
. ~/ros2_dashing/install/local_setup.bash
ros2 run demo_nodes_py listener
```

* Filesystem will be expanded to fit your SD Card size on first boot.
