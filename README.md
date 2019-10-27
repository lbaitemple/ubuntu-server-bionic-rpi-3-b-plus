# Ubuntu Server 18.04.2 (Bionic) ARM image for Raspberry Pi 3 B+
### Download image
https://github.com/AbelCS/ubuntu-server-bionic-rpi-3-b-plus/releases


### Instructions

Burn the image to SD card with dd/etcher/DiskWritter or your favorite tool.

Username: **ubuntu**  
Password: **ubuntu**

SSH is enabled by default, so you can login directly after first boot. After login, you can use the following commands
```
sudo apt-get update
sudo apt-get upgrade -y
```
If any lock files presented, please sudo remove them and 
```
sudo dpkg --configure -a
```
When everything is completed, you will need to do the following commands to enable I2C
```
sudo apt-get install python-pip python-pil  i2c-tools mosquitto-clients -y

```

### Notes

* To get wireless connection working on boot you must edit **/etc/netplan/01-rpi-3-network.yaml** present in *cloudimg-rootfs* partition in your sdcard and add your SSID and PASSWORD.

* Filesystem will be expanded to fit your SD Card size on first boot.
