Debian installer CD for Mini-PC
===============================

Introduction
------------

This is a set of configuration files for building a Debian
installation CD, optimized for "headless" mini-PC. It is based on the
installer for [PC-Engines APU (currently
discontinued)](https://github.com/ssinyagin/pcengines-apu-debian-cd).

Hardware and environment requirements:

* An Intel x86-64 compatiple CPU, such as Intel N100.

* COM0 serial port for the console.

* NVME PCI storage (the installer uses `/dev/nvme0n1`)

* One or more Ethernet NICs (the installer uses `enp1s0`), connected
  to a LAN with DHCP and Internet access.

* HDMI or DisplayPort display and a USB keyboard for the initial setup.

**WARNING: the installer erases everything on the NVME drive.**


The build machine MUST be of the same Debian release as the image that
you are building: a Bookworm host is required to produce a Bookworm
installer CD.

Before using these scripts, have a look at the files in profiles/, and
you may want to change some options.


"minipc" profile
---------------

This profile is designed to automate as much as possible.

* The default hostname is set to `minipc`, unless DHCP delivers a
different name. The installer prompts the user to enter the hostname
(or confirm the proposed name), and it's the only thing that needs a
user inoput. Default root password is `minipc`.

* `tty1` (HDMI console) and `ttyS0` (COM0) are both set as Linux
  console, and root can only login on them with a password.

* Disk layout:

  * 512MB EFI partition
  * 512MB /boot partition
  * the rest is the root partition

* The disk layout does not allocate a swap partition. Also, the system
is optimized for minimal swap usage: `vm.swappiness=1` is set in
`/etc/sysctl.d/minipc.conf`.

* The profile adds an hourly cronjob
(/etc/cron.hourly/minipc_fstrim) which performs fstrim command on
root and boot partitions once a day.

* The SSH daemon has `PermitRootLogin without-password` by default,
which disallows the root to login with a password. You either need to
install your public SSH key in root's autorized_keys, or change the SSH
daemon configuration.



Installation
------------

**WARNING**: this is a fully automated installation. Once you boot
from the USB stick, it will only give you 5 seconds in the initial
menu, then in a couple of minutes it will ask for the hostname, and
the rest will be done automatically, and a new Debian system will be
installed on the NVME drive.

**!!! ALL EXISTING DATA ON THE DRIVE WILL BE LOST !!!**

Both the build machine and the target mini-PC need the Internet
connection during the installation. The installer assumes that enp1s0
(marked as ETH0 on some enclosures) is connected to a network with
DHCP service and Internet access.

The installation ISO image is about 350MB in size.

Create the CD image on the build machine:

```
## install prerequisites
apt-get update
apt-get install -y simple-cdd git xorriso lsb-release mtools

## get the installer
git clone https://github.com/ssinyagin/minipc-debian-cd.git /opt/minipc-debian-cd

## build the installation CD image. 

cd /opt/minipc-debian-cd
./build minipc
```

In case of a failure, delete the contents of tmp/ subfolder.
Sometimes problems with accessing the mirror leave garbage in tmp/ and
this breaks the execution of build-simple-cdd

Insert the USB stick into the build machine

Check where your USB stick is:
```
lsblk
```

Copy the installer CD image to the USB stick:
```
dd if=images/debian-12-amd64-CD-1.iso of=/dev/sdc bs=16M
```

Insert the USB stick into the mini-PC, go into BIOS settings at the
start, and instruct it to boot from the USB stick.

The installation starts automatically, and it will ask for a hostname
within a couple of minutes. Then it will continue, and halt the
mini-PC when the installation finishes.

Start the mini-PC and it will boot the freshly installed Debian
system. You can access it from either a serial terminal at 115200
baud, or from the HDMI console and a USB keyboard.





Author
------

Stanislav Sinyagin

ssinyagin@k-open.com

