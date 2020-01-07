# GPU Passthrough

This text describes how to get GPU passthrough in KVM. I am using NVIDIA GPU, 
and the guys at NVidia has locked the drivers so they will not work in an virtual environment. 
Unless you pay big $ for their V100, you are stcuk with DirextX 3D accelleration layer.
VMWare always obey the big companies $$ fences (you know you can run Max OSX from a PC, with only a small patch, right?)

So the solution seems to be KVM (and that's not at all a poor decision).

This guide is written in a TLDR format. The sources can be found here:

[Installing KVM on Ubuntu](https://help.ubuntu.com/community/KVM/Installation)


# 1. Prerequisits

Note: This guide was written using Ubuntu 18.04 - 19.10

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

# 3. IOMMU and VFIO

## a) Enable IOMMU
edit the /etc/default/grub file and add **intel_iommu=on** to the GRUB_CMDLINE_LINUX_DEFAULT line.

```
GRUB_CMDLINE_LINUX_DEFAULT="nomodset intel_iommu=on"
```

then make this change visible to grub at boot by:
```
$ sudo update-grub
```

Now, reboot. After a successful reboot run this command

```
$ dmesg | grep IOMMU
```
This should list something like:
```
[0.141287] DMAR: IOMMU enabled
[0.241014] DMAR-IR: IOAPIC id 2 under DRHD base  0xfed90000 IOMMU 0
```
If the output is empty, then IOMMU is not enabled in kernel, which is strange since you must have manually disabled it. Make sure the new kernel from step 4 is successfully applied before continuing.

Now check IOMMU grouping by running this command:

```
$ for d in /sys/kernel/iommu_groups/*/devices/*; do n=${d#*/iommu_groups/*}; n=${n%%/*}; printf 'IOMMU Group %s ' "$n"; lspci -nns "${d##*/}"; done;
```

My system returns this:
```
IOMMU Group 0 00:00.0 Host bridge [0600]: Intel Corporation 8th Gen Core Processor Host Bridge/DRAM Registers [8086:3ec2] (rev 07)
IOMMU Group 10 00:1d.0 PCI bridge [0604]: Intel Corporation 200 Series PCH PCI Express Root Port #9 [8086:a298] (rev f0)
IOMMU Group 11 00:1d.4 PCI bridge [0604]: Intel Corporation 200 Series PCH PCI Express Root Port #13 [8086:a29c] (rev f0)
IOMMU Group 12 00:1f.0 ISA bridge [0601]: Intel Corporation Z370 Chipset LPC/eSPI Controller [8086:a2c9]
IOMMU Group 12 00:1f.2 Memory controller [0580]: Intel Corporation 200 Series/Z370 Chipset Family Power Management Controller [8086:a2a1]
IOMMU Group 12 00:1f.3 Audio device [0403]: Intel Corporation 200 Series PCH HD Audio [8086:a2f0]
IOMMU Group 12 00:1f.4 SMBus [0c05]: Intel Corporation 200 Series/Z370 Chipset Family SMBus Controller [8086:a2a3]
IOMMU Group 13 00:1f.6 Ethernet controller [0200]: Intel Corporation Ethernet Connection (2) I219-V [8086:15b8]
IOMMU Group 14 03:00.0 Non-Volatile memory controller [0108]: Samsung Electronics Co Ltd NVMe SSD Controller SM981/PM981/PM983 [144d:a808]
IOMMU Group 15 05:00.0 SATA controller [0106]: ASMedia Technology Inc. ASM1062 Serial ATA Controller [1b21:0612] (rev 02)
IOMMU Group 16 06:00.0 USB controller [0c03]: ASMedia Technology Inc. ASM2142 USB 3.1 Host Controller [1b21:2142]
IOMMU Group 17 08:00.0 Non-Volatile memory controller [0108]: Samsung Electronics Co Ltd NVMe SSD Controller SM981/PM981/PM983 [144d:a808]
IOMMU Group 18 09:00.0 USB controller [0c03]: NEC Corporation uPD720200 USB 3.0 Host Controller [1033:0194] (rev 03)
IOMMU Group 1 00:01.0 PCI bridge [0604]: Intel Corporation Xeon E3-1200 v5/E3-1500 v5/6th Gen Core Processor PCIe Controller (x16) [8086:1901] (rev 07)
IOMMU Group 1 00:01.1 PCI bridge [0604]: Intel Corporation Xeon E3-1200 v5/E3-1500 v5/6th Gen Core Processor PCIe Controller (x8) [8086:1905] (rev 07)
IOMMU Group 1 01:00.0 VGA compatible controller [0300]: NVIDIA Corporation TU102 [GeForce RTX 2080 Ti] [10de:1e04] (rev a1)
IOMMU Group 1 01:00.1 Audio device [0403]: NVIDIA Corporation TU102 High Definition Audio Controller [10de:10f7] (rev a1)
IOMMU Group 1 01:00.2 USB controller [0c03]: NVIDIA Corporation TU102 USB 3.1 Controller [10de:1ad6] (rev a1)
IOMMU Group 1 01:00.3 Serial bus controller [0c80]: NVIDIA Corporation TU102 UCSI Controller [10de:1ad7] (rev a1)
IOMMU Group 1 02:00.0 VGA compatible controller [0300]: NVIDIA Corporation TU106 [GeForce RTX 2060 Rev. A] [10de:1f08] (rev a1)
IOMMU Group 1 02:00.1 Audio device [0403]: NVIDIA Corporation TU106 High Definition Audio Controller [10de:10f9] (rev a1)
IOMMU Group 1 02:00.2 USB controller [0c03]: NVIDIA Corporation TU106 USB 3.1 Host Controller [10de:1ada] (rev a1)
IOMMU Group 1 02:00.3 Serial bus controller [0c80]: NVIDIA Corporation TU106 USB Type-C Port Policy Controller [10de:1adb] (rev a1)
IOMMU Group 2 00:14.0 USB controller [0c03]: Intel Corporation 200 Series/Z370 Chipset Family USB 3.0 xHCI Controller [8086:a2af]
IOMMU Group 2 00:14.2 Signal processing controller [1180]: Intel Corporation 200 Series PCH Thermal Subsystem [8086:a2b1]
IOMMU Group 3 00:16.0 Communication controller [0780]: Intel Corporation 200 Series PCH CSME HECI #1 [8086:a2ba]
IOMMU Group 4 00:17.0 SATA controller [0106]: Intel Corporation 200 Series PCH SATA controller [AHCI mode] [8086:a282]
IOMMU Group 5 00:1b.0 PCI bridge [0604]: Intel Corporation 200 Series PCH PCI Express Root Port #17 [8086:a2e7] (rev f0)
IOMMU Group 6 00:1c.0 PCI bridge [0604]: Intel Corporation 200 Series PCH PCI Express Root Port #1 [8086:a290] (rev f0)
IOMMU Group 7 00:1c.1 PCI bridge [0604]: Intel Corporation 200 Series PCH PCI Express Root Port #2 [8086:a291] (rev f0)
IOMMU Group 8 00:1c.4 PCI bridge [0604]: Intel Corporation 200 Series PCH PCI Express Root Port #5 [8086:a294] (rev f0)
IOMMU Group 9 00:1c.7 PCI bridge [0604]: Intel Corporation 200 Series PCH PCI Express Root Port #8 [8086:a297] (rev f0)

```

As you will notice, I have two GPUs and they was put in the same group: IOMMU Group 1. This is probably good for SLI performance, but bad for GPU passthroug (unless you will pass them both to KVM). I will have to split the group so each card is in its own group. This can be done by changing the PCI slot in some motherboards, but I will use ACS hack for this.
It is really important that the pass through GPU is in its OWN IOMMU group, not shared with ANY other device. Also remember that newer NVIDIA cards have three devices: VGA, Audio and USB. These 3 must be passed throu together, and will always be in the same group. Also some motherboards, like mine, adds the physical PCI slot as well. You can just ignore that device (it the PCI Bridge(s) in Group 1)

Things to try that may split separate the GPU from other devices:

a) Move to a different PCI-e slot.
b) Make sure you have the latest kernel for your build (newer than 4.19)
c) Update bios firmware on the motherboard

If you have tried all options, and the GPU still shares IOMMU Group with other devices, then you will need the ACS patch from the next step. If the target GPU is in its own group, skip to step 5.


# 4. Patched kernel for ACS support

This step is optional, and must only be applied if :


You have two options: 
 1) Download and install a pre-patched kernel or 
 2) Download the kernel sources and build your own kernel
 
## Option a) Pre-patched kernel
 Download the kernel from here: [ACS Patch]https://queuecumber.gitlab.io/linux-acs-override/
 Choose the Kernel that best matches your running kernel. To see running kernel version:
 ```
$ uname -a
```
 The output would be something like: 
 
 *Linux mymachine 5.3.0-24-common #2 SMP Fri Jan 3 16:22:32 CET 2020 x86_64 x86_64 x86_64 GNU/Linux*
 
 The '5.3.0-24-common' text is the version. You should then download the latest 5.3.x kernel
 
 Install the downloaded kernel by running:
 
 ```
$ sudo dpkg -i linux-image-5.3.18-acso_5.3.18-acso-1_amd64.deb
```

This example is for the 5.3.18 kernel
 
 
## Option b) Build you own kernel
This requires some preinstalled tools and libraries
To be completed ...


## Enabling ACS

edit the /etc/default/grub file and add **pcie_acs_override=downstream** to the GRUB_CMDLINE_LINUX_DEFAULT line after the intel_iommu from step 3.a.

```
GRUB_CMDLINE_LINUX_DEFAULT="nomodset intel_iommu=on pcie_acs_override=downstream"
```

then make this change visible to grub at boot by:
```
$ sudo update-grub
```

Now, reboot. After a successful reboot check IOMMU group assignments with:
```
$ for d in /sys/kernel/iommu_groups/*/devices/*; do n=${d#*/iommu_groups/*}; n=${n%%/*}; printf 'IOMMU Group %s ' "$n"; lspci -nns "${d##*/}"; done;
```

Check that your devices are split into separate groups. My cards ended up in groups 15 and 16.
```
...
IOMMU Group 15 01:00.0 VGA compatible controller [0300]: NVIDIA Corporation TU102 [GeForce RTX 2080 Ti] [10de:1e04] (rev a1)
IOMMU Group 15 01:00.1 Audio device [0403]: NVIDIA Corporation TU102 High Definition Audio Controller [10de:10f7] (rev a1)
IOMMU Group 15 01:00.2 USB controller [0c03]: NVIDIA Corporation TU102 USB 3.1 Controller [10de:1ad6] (rev a1)
IOMMU Group 15 01:00.3 Serial bus controller [0c80]: NVIDIA Corporation TU102 UCSI Controller [10de:1ad7] (rev a1)
IOMMU Group 16 02:00.0 VGA compatible controller [0300]: NVIDIA Corporation TU106 [GeForce RTX 2060 Rev. A] [10de:1f08] (rev a1)
IOMMU Group 16 02:00.1 Audio device [0403]: NVIDIA Corporation TU106 High Definition Audio Controller [10de:10f9] (rev a1)
IOMMU Group 16 02:00.2 USB controller [0c03]: NVIDIA Corporation TU106 USB 3.1 Host Controller [10de:1ada] (rev a1)
IOMMU Group 16 02:00.3 Serial bus controller [0c80]: NVIDIA Corporation TU106 USB Type-C Port Policy Controller [10de:1adb] (rev a1)
I
...
```

I will pass through group 16 (My 2060 card), so I take note of the three device ids: 10de:10f9, 10de:1ada and 10de:1adb as I will use them in the next step

# 5. Applying VFIO

What is VFIO, and why do I need it?
Shortly said: Virual Function I/O allows a VM direct access to a device. Without it the running kernel on the host machine will claim the device, and it cannot be passed through.

## a) Get device IDs for the devices you want to pass through
You need to find the device IDs for the GPU (and the GPUs Audio and USB device) you want to pass through.
First get bus ID:
```
$ lspci
```

The output lists connected devices:
```
00:1f.2 Memory controller: Intel Corporation 200 Series/Z370 Chipset Family Power Management Controller
00:1f.3 Audio device: Intel Corporation 200 Series PCH HD Audio
00:1f.4 SMBus: Intel Corporation 200 Series/Z370 Chipset Family SMBus Controller
00:1f.6 Ethernet controller: Intel Corporation Ethernet Connection (2) I219-V
01:00.0 VGA compatible controller: NVIDIA Corporation TU102 [GeForce RTX 2080 Ti] (rev a1)
01:00.1 Audio device: NVIDIA Corporation TU102 High Definition Audio Controller (rev a1)
01:00.2 USB controller: NVIDIA Corporation TU102 USB 3.1 Controller (rev a1)
01:00.3 Serial bus controller [0c80]: NVIDIA Corporation TU102 UCSI Controller (rev a1)
02:00.0 VGA compatible controller: NVIDIA Corporation TU106 [GeForce RTX 2060 Rev. A] (rev a1)
02:00.1 Audio device: NVIDIA Corporation TU106 High Definition Audio Controller (rev a1)
02:00.2 USB controller: NVIDIA Corporation TU106 USB 3.1 Host Controller (rev a1)
02:00.3 Serial bus controller [0c80]: NVIDIA Corporation TU106 USB Type-C Port Policy Controller (rev a1)
03:00.0 Non-Volatile memory controller: Samsung Electronics Co Ltd NVMe SSD Controller SM981/PM981/PM983
05:00.0 SATA controller: ASMedia Technology Inc. ASM1062 Serial ATA Controller (rev 02)
06:00.0 USB controller: ASMedia Technology Inc. ASM2142 USB 3.1 Host Controller
08:00.0 Non-Volatile memory controller: Samsung Electronics Co Ltd NVMe SSD Controller SM981/PM981/PM983
09:00.0 USB controller: NEC Corporation uPD720200 USB 3.0 Host Controller (rev 03)
```
I want to use my 2060 card, so I note 02:00

Then list device IDS:
```
$ lspci -vn -s 02:00
```
For my system it outputs these 4 devices:

```
02:00.0 0300: 10de:1f08 (rev a1) (prog-if 00 [VGA controller])
	Subsystem: 1458:3fc2
	Flags: bus master, fast devsel, latency 0, IRQ 183
	Memory at dc000000 (32-bit, non-prefetchable) [size=16M]
	Memory at a0000000 (64-bit, prefetchable) [size=256M]
	Memory at b0000000 (64-bit, prefetchable) [size=32M]
	I/O ports at d000 [size=128]
	[virtual] Expansion ROM at dd000000 [disabled] [size=512K]
	Capabilities: <access denied>
	Kernel driver in use: nvidia
	Kernel modules: nvidiafb, nouveau, nvidia_drm, nvidia

02:00.1 0403: 10de:10f9 (rev a1)
	Subsystem: 1458:3fc2
	Flags: bus master, fast devsel, latency 0, IRQ 18
	Memory at dd080000 (32-bit, non-prefetchable) [size=16K]
	Capabilities: <access denied>
	Kernel driver in use: snd_hda_intel
	Kernel modules: snd_hda_intel

02:00.2 0c03: 10de:1ada (rev a1) (prog-if 30 [XHCI])
	Subsystem: 1458:3fc2
	Flags: fast devsel, IRQ 132
	Memory at b2000000 (64-bit, prefetchable) [size=256K]
	Memory at b2040000 (64-bit, prefetchable) [size=64K]
	Capabilities: <access denied>
	Kernel driver in use: xhci_hcd

02:00.3 0c80: 10de:1adb (rev a1)
	Subsystem: 1458:3fc2
	Flags: bus master, fast devsel, latency 0, IRQ 150
	Memory at dd084000 (32-bit, non-prefetchable) [size=4K]
	Capabilities: <access denied>
	Kernel driver in use: nvidia-gpu
	Kernel modules: i2c_nvidia_gpu

```

This would be the GPU itself, Audio device, USB-C, and serial bus controller attached to the discreet GPU card in slot 2
All of these devices should be passed through, and added to VFIO.

Device ids: 10de:1f08, 10de:1f09, 10de:1ada, 10de:1adb

## b) Configuring VFIO

### Edit the /etc/initramfs-tools/modules:
```
$ sudo nano /etc/initramfs-tools/modules
```

Add this line:
vfio vfio_iommu_type1 vfio_virqfd vfio_pci ids=<your ids from step 5.a>

My file now looks like this:
```
# List of modules that you want to include in your initramfs.
# They will be loaded at boot time in the order below.
#
# Syntax:  module_name [args ...]
#
# You must run update-initramfs(8) to effect this change.
#
# Examples:
#
# raid1
# sd_mod
vfio 
vfio_iommu_type1 
vfio_virqfd 
vfio_pci ids=10de:1f08,10de:1f09,10de:1ada,10de:1adb
vfio_pci

```

### Edit the /etc/modules file and add vfio vfio_iommu_type1 vfio_pci ids=<your device list>

My file looks like this:
```
# /etc/modules: kernel modules to load at boot time.
#
# This file contains the names of kernel modules that should be loaded
# at boot time, one per line. Lines beginning with "#" are ignored.
vfio 
vfio_iommu_type1 
vfio_pci ids=10de:1f08,10de:1f09,10de:1ada,10de:1adb
```

### Module loading order
It is really important that the device is passed to VFIO *before* any host drivers claim it. To ensure this, create a file called nvidia.conf in /etc/modprobe.d:

File content of /etc/modprobe.d/nvidia.conf:
```
softdep nouveau pre: vfio-pci 
softdep nvidia pre: vfio-pci 
softdep nvidia* pre: vfio-pci
```


### VFIO Config file
To tell VFIO which devices to use, create a new file /etc/modprobe.d/vfio.conf with this content:
```
# VFIO config file
options vfio-pci ids=10de:1f08,10de:1f09,10de:1ada,10de:1adb
```

### KVM config

If you plan to run Windows 10 18.03 or later, you must add this line to /etc/modprobe.d/kvm.conf:
```
options kvm ignore_msrs=1
```
Create a new file if it doen not exist


### Update initramfs

To apply all these changes to boot time, rune the following command:
```
$ sudo update-initramfs -u -k all
```

## Reboot

Everything should now be set for a successfull GPU pass through. It is time to reboot, but remember: If you don't have a second graphics card in the machine, you will have NO OUTPUT. The kernel will not try to load a driver for the devices you put in the vfio.conf file




