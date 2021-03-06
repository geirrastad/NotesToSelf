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
NOTE: The above did not survive a boot on my system! I spent a couple of hours
banging my head beacuse GPSD did not work. Then I happened to see "agetty" 
claiming the ttyAMA0 device in "ps -ef". I dont know how to completely disable this, yet.

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
$ sudo cat /dev/ttyAMA0
```


3.6) Setting up PPS support

Install the PPS tools
```
$ sudo apt-get install pps-tools
```

3.7) Enable PPS in the kernel

Edit /boot/config.txt and add these lines at the bottom:
```
# enable GPS PPS
dtoverlay=pps-gpio,gpiopin=4
```

Reboot to test PPS.

3.8) Disable NTP support in DHCP

Edit the /etc/dhcp/dhclient.conf, and remove the *"ntp-servers"* from the *request* string
```
request subnet-mask, broadcast-address, time-offset, routers,
        domain-name, domain-name-servers, domain-search, host-name,
        dhcp6.name-servers, dhcp6.domain-search,
        netbios-name-servers, netbios-scope, interface-mtu,
        rfc3442-classless-static-routes , *ntp-servers*;
```

Also remove or comment out any "*option ntp_servers*" in that file!

 The delete these files:
/etc/dhcp/dhclient-exit-hooks.d/ntp || /etc/dhcp/dhclient-exit-hooks.d/timesyncd
/lib/dhcpcd/dhcpcd-hooks/50-ntp.conf
/var/lib/ntp/ntp.conf.dhcp (might not exist)


3.10 ) Reboot!


## 4. Setting up GPSD

4.1) Install GPSD
```
$ sudo apt install gpsd gpsd-clients
```

Edit the /etc/defaults/gpsd and make sure it looks like this:
```
# Default settings for the gpsd init script and the hotplug wrapper.

# Start the gpsd daemon automatically at boot time
START_DAEMON="true"

# Use USB hotplugging to add new USB devices automatically to the daemon
USBAUTO="false"

# Devices gpsd should collect to at boot time.
# They need to be read/writeable, either by user gpsd or the group dialout.
DEVICES="/dev/ttyAMA0 /dev/pps0"

# Other options you want to pass to gpsd
GPSD_OPTIONS="-n"
```

4.2) Configuring sysctl handlers

```
$ sudo systemctl daemon-reload
$ sudo systemctl enable gpsd
$ sudo systemctl start gpsd
```

4.3) Check status
```
$ systemctl status gpsd
● gpsd.service - GPS (Global Positioning System) Daemon
   Loaded: loaded (/lib/systemd/system/gpsd.service; indirect; vendor preset: enabled)
   Active: active (running) since Fri 2019-01-11 15:43:59 CET; 3min 45s ago
 Main PID: 747 (gpsd)
   CGroup: /system.slice/gpsd.service
           └─747 /usr/sbin/gpsd -N

Jan 11 15:43:59 ntpserver01 systemd[1]: Started GPS (Global Positioning System) Daemon.

```

## 5. New NTP version

It seems the NTP packaged for raspi does not support PPS out of the box. Download and install a new NTP
```
mkdir ~/ntp
cd ~/ntp
sudo apt install libcap-dev libssl-dev
wget http://www.eecis.udel.edu/~ntp/ntp_spool/ntp4/ntp-4.2/ntp-4.2.8p12.tar.gz
tar -xzvf ntp-4.2.8p12.tar.gz 
cd ntp-4.2.8p12
./configure --enable-linuxcaps
make -j5
sudo apt-get remove ntp
sudo make install 

```
