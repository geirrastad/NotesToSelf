# GPU Passthrough

This text describes how to get GPU passthrough in KVM. I am using NVIDIA GPU, 
and the guys at NVidia has locked the drivers so they will not work in an virtual environment. 
Unless you pay big $ for their V100, you are stcuk with DirextX 3D accelleration layer.
VMWare always obey the big companies $$ fences (you know you can run Max OSX from a PC, with only a small patch, right?)

So the solution seems to be KVM (and that's not at all a poor decision).

This guide is written in a TLDR format. The sources can be found here:

[Installing KVM on Ubuntu](https://help.ubuntu.com/community/KVM/Installation)


# 1. Prerequisits


## a) Check KVM support
```
$ egrep -c '(vmx|svm)' /proc/cpuinfo
```
It should return >4. Mine returned 12 (i7 8086K CPU)

```
$ kvm-ok
```
Shoukld return something similar to:
```
INFO: /dev/kvm exists
KVM acceleration can be used
```

## b) Memory and stuff
To server more than 2 Gb ram to VM's you MUST run a 64 bit kernel.

To check for 64bit: 
```
$ egrep -c ' lm ' /proc/cpuinfo
```
It should again return more than 0 (number of CPU cores supporting long mode a.k.a 64 bit)


# 2. Install KVM

Note: This guide is for Ubuntu 18.04

## a) Install Software and dependencies

```
sudo apt-get install qemu-kvm libvirt-bin ubuntu-vm-builder bridge-utils
```
For 18.10:
```
$ sudo apt-get install qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils
```


## b) Add user to libvrt

```
$ sudo adduser `id -un` libvirt
```
It may say that user is already added, and that is fine!

## c) Check installation

To see if everything is OK, you should log out, then back in after completing step b)
Now run this:
```
$ virsh list --all
```

and it should output something like:
```
 Id    Name                           State
----------------------------------------------------

```

If it didn't something went wrong and you will have to redo all steps...
