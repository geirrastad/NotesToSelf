# Create a master NTP server using Raspberry

To allow correct timestamping on all events in our systems, we rely on a local GPS+PPS enabled time server.
I have created this using a Raspberry Pi 3 B+, Adafruit GPS HAT, External GPS Antenna and the NTP & GPSD
software as a basis. Then syncing all servers to this NTP server. This is an isolated network with NO internet connection,
so the NTP server must only use GPS/PPS as a source.

However, a correct timestamp on servers is not enough to be able to correctly timestamp all events. You will have to consider
network delay, kernel call delays, API call delays etc. All these delays wil make the timestamp too inaccurate 
(for our needs, others may find this perfeclty tolerable). 

# 1. Getting started

This guide assumes you have sufficient knowledge of Linux and Raspbian.

1.1) Download and install Raspbian Lite (latest version)
1.2) Add the file "ssh" to the boot parttion on the SD-Card so SSH is enabled at boot.
1.3) Log in to your device


# 2. GPS Hat

Solder and fit the GPS hat. Also add an external antenna if you need.

