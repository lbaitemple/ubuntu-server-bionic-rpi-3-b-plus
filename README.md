# Ubuntu Server 18.04.2 (Bionic) ARM image for Raspberry Pi 3 B+
### Download image
https://github.com/AbelCS/ubuntu-server-bionic-rpi-3-b-plus/releases


### Instructions

Burn the image to SD card with dd/etcher/DiskWritter or your favorite tool.

Username: **ubuntu**  
Password: **ubuntu**

SSH is enabled by default, so you can login directly after first boot. It is better to setup everything on the same network so you can ssh into the pi. After login, you can use the following commands by copying and pasting into the ssh shell
```
sudo apt-get update
sudo apt-get upgrade -y
```
If any lock files presented, please sudo remove them and 
```
sudo rm /var/lib/dpkg/lock*
sudo dpkg --configure -a
```
When everything is completed, you will need to do the following commands to enable I2C
```
sudo apt-get install python-pip python-pil  i2c-tools mosquitto-clients -y
sudo pip install Adafruit_SSD1306 RPi.GPIO
```
### set up I2C
To enable I2c permission
```
sudo chgrp i2c /dev/i2c-1
sudo chmod 666 /dev/i2c-1
sudo usermod -G i2c $USER
```

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


### Setup everything for IP 

* To get wireless connection working on boot you must edit **/etc/netplan/01-rpi-3-network.yaml** present in *cloudimg-rootfs* partition in your sdcard and add your SSID and PASSWORD.
Open file
```
sudo nano /etc/netplan/01-rpi-3-network.yaml
```
Please include the following content and make sure you generate your password hash (see line 84)
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
                                   password: hash:[echo -n [password] | iconv -t UTF-16LE | openssl md4 ]
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

You can open test2.sh and modify cloud MQTT setting
```
sudo nano ~/test2.sh
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


### Install ROS
```
sudo sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'
sudo apt-key adv --keyserver hkp://ha.pool.sks-keyservers.net:80 --recv-key C1CF6E31E6BADE8868B172B4F42ED6FBAB17C654
sudo apt-get update
sudo apt-get install ros-melodic-desktop -y
sudo rosdep init
rosdep update
echo "source /opt/ros/melodic/setup.bash" >> ~/.bashrc
```

* Filesystem will be expanded to fit your SD Card size on first boot.
