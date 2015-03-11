![Cirrus Logic Logo](https://github.com/CirrusLogic/wiki-content/blob/master/logos/Cirrus%20Logic%20286.png)

# Building the code for the Element 14 Cirrus audio card

Last updated : Tuesday, 10 March 2015

This tutorial shows how to build a Raspberry PI kernel and install it on to your device. This tutorial assumes that you are building on a Linux machine as building on the Raspberry PI nativily is very slow.

Note: This has been tested with the lastest updates to Raspian as of the time of writing.
Download Raspian from [here](http://www.raspberrypi.org/downloads/).
To install Raspian follow the instructions [here](http://www.raspberrypi.org/documentation/installation/installing-images/README.md).

1) Download the Raspberry PI cross-compilers:

```
cd ~/bin
mkdir raspberrypi
cd raspberrypi
git clone https://github.com/raspberrypi/tools
```

2) Set CCPREFIX environment variable:
```
export CCPREFIX=/home/<user>/raspberrypi/tools/arm-bcm2708/gcc-linaro-arm-linux-gnueabihf-raspbian/bin/arm-linux-gnueabihf-
${CCPREFIX}gcc -v
```
Note : Change ```<user>``` to your username

3) Download the Cirrus Raspberry PI kernel
```
cd ~
mkdir code
cd code
git clone https://github.com/CirrusLogic/rpi-linux.git
cd rpi-linux
```


4) Create a .config file that includes the audio card code.

when the target platform is bcm2708 i.e. Raspberry Pi boards
```
make ARCH=arm bcmrpi_defconfig
```
when the target platform is bcm2709 i.e. Raspberry Pi 2 boards
```
make ARCH=arm bcm2709_defconfig
```
You could be asked a lot of questions but you can answer the default to all of these.

5) Build the kernel and kernel modules
```
ARCH=arm CROSS_COMPILE=${CCPREFIX} make -j <num-cores>
ARCH=arm CROSS_COMPILE=${CCPREFIX} INSTALL_MOD_PATH=../modules make modules_install
```
Note : Where ```<num-cores>``` is the number of processor cores dedicated to the build

6) Upload the new kernel, modules and dtb file to the Raspberry PI
```
cd /home/<user>/raspberrypi/tools/mkimage
./mkknlimg  ~/code/rpi-linux/arch/arm/boot/zImage <kernel_img>
scp <kernel_img> pi@<ip-addr>:/tmp
cd ~/code/modules
tar czf modules.tgz *
scp modules.tgz pi@<ip-addr>:/tmp
scp ~/code/arch/arm/boot/dts/rpi-cirrus-wm5102-overlay.dtb pi@<ip-addr>:/tmp
```
Note : Where ```<kernel_img>``` is '''kernel.img''' for bcm2708, '''kernel7.img''' for bcm2709

7) Install the kernel, modules and dtb file on the Raspberry PI. Connect to your
Raspberry PI over SSH and do the following:
```
cd /
sudo mv /tmp/<kernel_img> /boot/
sudo tar xzf /tmp/modules.tgz
sudo mv /tmp/rpi-cirrus-wm5102-overlay.dtb /boot/overlays/
rm /tmp/modules.tgz
```

8) Device tree configuration for cirrus audio card
Open the '''config.txt''' file and add entry for '''dtoverlay'''
'''
sudo nano /boot/config.txt
dtoverlay=rpi-cirrus-wm5102-overlay
'''

9) Dependancies between modules. There are certain modules those should be loaded before loading of other modules.
Connect to the Raspberry PI over SSH and do the following:
```
sudo nano /etc/modprobe.d/raspi-blacklist.conf
```
Note : create ```raspi-blacklist.conf``` if it is not existed

Add the following lines to the file
```
softdep arizona-spi pre: arizona-ld
softdep spi-bcm2708 pre: fixed
```

10) Restart the Raspberry PI
'''
sudo shutdown -r now
'''

11) When the Raspberry PI reboots connect over SSH and run:
```
uname -a
```
This will display the new kernel version.
