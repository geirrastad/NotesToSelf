# Create a master NTP server using Raspberry

To allow correct timestamping on all events in our systems, we rely on a local GPS+PPS enabled time server.
I have created this using a Raspberry Pi 3 B+, Adafruit GPS HAT, External GPS Antenna and the NTP & GPSD
software as a basis. Then syncing all servers to this NTP server. This is an isolated network with NO internet connection,
so the NTP server must only use GPS/PPS as a source.

However, a correct timestamp on servers is not enough to be able to correctly timestamp all events. You will have to consider
network delay, kernel call delays, API call delays etc. All these delays wil make the timestamp too inaccurate 
(for our needs, others may find this perfeclty tolerable). 

## 1. Getting started

This guide assumes you have sufficient knowledge of Linux and Raspbian.

1.1) Download and install Raspbian Lite (latest version)

1.2) Add the file "ssh" to the boot parttion on the SD-Card so SSH is enabled at boot.

1.3) Log in to your device. If you only have this PI connected to your net, you may reach it by:
```
$ ssh raspberrypi
```
This, ofcourse, depends on your DHCP and network settings. You could also look at your router to see what addres it got.


## 2. GPS Hat

NOTE: Using the GPS Hat disables bluetooth, and UART on your Raspberry. You must login using SSH or usb keyboard / HDMI display.

Solder and fit the GPS hat. Also add an external antenna if you need. You should really fit the CS1220 battery as 
it will speed up recovery time after cold reboot or power loss.

## 3. Setting up the PI

This chapter is based on [this guide by Steve Friedl](http://unixwiz.net/techtips/raspberry-pi3-gps-time.html)

3.1) Update your pi
```
$ sudo apt update
$ sudo apt upgrade
$ sudo reboot
```

3.2) Do other customizations to your PI. Language, keyboard, new password etc
```
$ sudo raspi-config
```

3.3) Disable getty

For the Ultimate GPS Hat to function on a Pi3, we need to disable and shuffle around with the hardware UART. 

```
$ sudo systemctl stop serial-getty@ttyAMA0.service
$ sudo systemctl disable serial-getty@ttyAMA0.service
```
Now edit the /boot/cmdline.txt file and remove the "console=serial0,115200"
Original file:
```
dwc_otg.lpm_enable=0 console=serial0,115200 console=tty1 root=PARTUUID=1...
```
After edit:
```
dwc_otg.lpm_enable=0 console=tty1 root=PARTUUID=1...
```

3.4) Disable Bluetooh and hijack hardware UART

Edit /boot/config.txt and add these lines at the bottom:
```
# Use the /dev/ttyAMA0 UART for user applications (GPS), not Bluetooth
dtoverlay=pi3-disable-bt
```
Now run:
```
$ sudo systemctl disable hciuart
```

reboot your PI

3.5) Checking NMEA data

If everything is OK, you should be able to see the NMEA data on /dev/ttyAMA0

```
$ cat /dev/ttyAMA0


```
