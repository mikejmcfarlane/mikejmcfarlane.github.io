---
layout: post
title: PXE boot a Raspberry Pi 4 from a Synology Diskstation and compare performance to microSD
date: 2020-09-12
---

- [Introduction](#introduction)
- [Setting up the Synology](#setting-up-the-synology)
  - [Setup manual IP for the Synology](#setup-manual-ip-for-the-synology)
  - [Enable NFS](#enable-nfs)
  - [Create shared folders and share them via NFS](#create-shared-folders-and-share-them-via-nfs)
  - [Enable the TFTP service](#enable-the-tftp-service)
  - [Setup DHCP](#setup-dhcp)
  - [Set the DHCP vendor setting](#set-the-dhcp-vendor-setting)
  - [Not quite finished with the Synology!](#not-quite-finished-with-the-synology)
- [Setting up the Raspberry Pi to PXE boot](#setting-up-the-raspberry-pi-to-pxe-boot)
  - [Set the hostname](#set-the-hostname)
  - [Copy the OS files to the rpi-pxe folder on the Synology](#copy-the-os-files-to-the-rpi-pxe-folder-on-the-synology)
  - [Copy the boot files to the rpi-tftpboot folder on the Synology](#copy-the-boot-files-to-the-rpi-tftpboot-folder-on-the-synology)
  - [Configure some boot options](#configure-some-boot-options)
  - [Configure the eeprom to include PXE boot options](#configure-the-eeprom-to-include-pxe-boot-options)
- [Final step on the Synology](#final-step-on-the-synology)
- [And PXE boot](#and-pxe-boot)
- [Debugging when things go wrong](#debugging-when-things-go-wrong)
- [Performance tests](#performance-tests)
  - [Write testing](#write-testing)
  - [Read testing](#read-testing)
  - [Comparison results](#comparison-results)
- [Next steps](#next-steps)
- [Resources I used to get this working](#resources-i-used-to-get-this-working)
  - [Setup](#setup)
  - [bootloader config](#bootloader-config)

## Introduction

PXE booting allow a server, in this case a Raspberry Pi 4, to boot from a remote computer, in this case a Synology Diskstation DS720+. I also did the same process with an older Synology Diskstation DS212+.

Why might you do this? There are some reasons I like:

1. microSD cards have a limited write lifetime, so moving the OS off the microSD card should be more reliable in the long run.
2. PXE boots are easier to backup, as can run a backup or snapshot with Synology HyperBackup.
3. PXE boots for multiple Pis that all require configured should be easier to automate than multiple SD cards that would each need written to on my Mac first.
4. Lastly, I just got the DS720+ and want to really try it out.

I'm going to outline the key steps I needed to do for the Raspberry Pi to PXE boot from the Synology. The key part for me that took some time to figure out was setting up the boot loader config, and once this was right things just worked.

Lastly I did some brief disk performance tests comparing microSD to a PXE boot on both Diskstations (with both a single Pi testing and 3 Pis testing simultaneously) to a USB 3.0 magnetic HDD drive.

## Setting up the Synology

The first step is to configure the Synology to act as a DHCP server and to configure it to serve up the the Raspberry Pi operating system files via [NFS](https://www.raspberrypi.org/documentation/configuration/nfs.md).

### Setup manual IP for the Synology

![manual_ip]({{ site.url }}/assets/manual_ip.png)

![manual_ip_setup]({{ site.url }}/assets/manual_ip_setup.png)

### Enable NFS

![enable_nfs]({{ site.url }}/assets/enable_nfs.png)

### Create shared folders and share them via NFS

Create two shared folders:

* rpi-pxe (used for the OS files)
* rpi-tftpboot (used for the boot files)

![shared_folders]({{ site.url }}/assets/shared_folders.png)

And then for each folder edit the NFS permissions:

![shared_folder_nfs_permissions_edit_rpi-pxe]({{ site.url }}/assets/shared_folder_nfs_permissions_edit_rpi-pxe.png)

![shared_folder_nfs_permissions_edit_rpi-tftpboot]({{ site.url }}/assets/shared_folder_nfs_permissions_edit_rpi-tftpboot.png)

### Enable the TFTP service

![tftp]({{ site.url }}/assets/tftp.png)

### Setup DHCP

![dhcp_network]({{ site.url }}/assets/dhcp_network.png)

![dhcp_edit]({{ site.url }}/assets/dhcp_edit.png)

![dhcp_subnet]({{ site.url }}/assets/dhcp_subnet.png)

### Set the DHCP vendor setting

![dhcp_vendor_setting]({{ site.url }}/assets/dhcp_vendor_setting.png)

### Not quite finished with the Synology!

There is one more step to do on the Synology and that is to enable PXE booting and define the boot loader file, but we can't do that till we have done some work on the Raspberry Pi, so it's over to the Pi we go.

## Setting up the Raspberry Pi to PXE boot

I'm going to assume you have done the basics of getting a Raspberry Pi booted from a microSD card, completed basic config and have SSH enabled or access to a terminal by connecting the Pi to a monitor and keyboard. I used Raspberry Pi OS (32-bit) Lite from August 2020, but I think the process should be similar for all Pi OSs and other recent versions.

### Set the hostname

If you have more than one Pi it is worth setting a unique hostname for each. Mine use the naming convention _ev3dev-piXX_.

```
sudo vi /etc/hostname
```
and enter a new hostname, then
```
sudo reboot
```
When you reconnect via SSH you should use the the new hostname.

### Copy the OS files to the rpi-pxe folder on the Synology

Firstly ensure you have the necessary files for NFS:
```
sudo apt install nfs-common
```

Create a new NFS mount directory on the Pi that matches it's hostname:
```
sudo mkdir /nfs/ev3dev-pi02
```
then mount the _rpi-pxe_ shared folder with NFS:
```
sudo mount -t nfs -O proto=tcp,port=2049,rw,all_squash,anonuid=1001,anongid=1001 192.168.1.191:/volume1/rpi-pxe/ev3dev-pi02 /nfs/ev3dev-pi02 -vvv
```
then finally copy the OS files:
```
sudo rsync -xa --progress --exclude /nfs / /nfs/ev3dev-pi02/
```

### Copy the boot files to the rpi-tftpboot folder on the Synology


Create a new NFS mount directory:
```
sudo mkdir -p /nfs/rpi-tftpboot
```
mount the _rpi-tftpboot_ shared folder with NFS:
```
sudo mount -t nfs -O proto=tcp,port=2049,rw,all_squash,anonuid=1001,anongid=1001 192.168.1.191:/volume1/rpi-tftpboot /nfs/rpi-tftpboot -vvv
```
Get the Pi serial number:
```
vcgencmd otp_dump | grep 28: | sed s/.*://g
```
Create a folder for the boot files based on the serial number:
```
sudo mkdir -p /nfs/rpi-tftpboot/d112bbc5
```
Lastly copy the boot files:
```
sudo cp -r /boot/* /nfs/rpi-tftpboot/d112bbc5/
```

### Configure some boot options

Firstly, edit _/etc/fstab_ to mount the boot folder at boot:
```
sudo vi /nfs/ev3dev-pi02/etc/fstab
```
and edit it to look like:
```
proc            /proc           proc    defaults          0       0
192.168.1.191:/volume1/rpi-tftpboot/d112bbc5 /boot nfs defaults,vers=3,proto=tcp 0 0
```
Secondly, edit the kernel options in _cmdline.txt_:
```
sudo vi /nfs/rpi-tftpboot/d112bbc5/cmdline.txt
```
and edit it to look like:
```
console=serial0,115200 console=tty1 root=/dev/nfs nfsroot=192.168.1.191:/volume1/rpi-pxe/ev3dev-pi02,vers=3 rw ip=dhcp elevator=deadline rootwait
```

### Configure the eeprom to include PXE boot options

This was a critical step for me. I need to ensure that _TFTP_IP_ and _BOOT_ORDER_ both had correct values.

Install software for working with the eeprom:
```
sudo apt install rpi-eeprom
```
then copy the lastest firmware file to a temporary directory in your home folder:
```
cd ~/tmp/
cp /lib/firmware/raspberrypi/bootloader/stable/pieeprom-2020-07-31.bin pieeprom.bin
```
If you have previously been through this process delete any re-written eeprom files, as the next step to create a new binary silently fails but still looks like you have written a new eeprom file:
```
rm pieeprom-new.bin
```
Create a boot configuration file:
```
vi bootconf.txt
```
and edit it to look like:
```
[all]
BOOT_UART=0
WAKE_ON_GPIO=1
POWER_OFF_ON_HALT=0
DHCP_TIMEOUT=45000
DHCP_REQ_TIMEOUT=4000
TFTP_FILE_TIMEOUT=30000
TFTP_IP=192.168.1.191
TFTP_PREFIX=0
ENABLE_SELF_UPDATE=1
DISABLE_HDMI=0
BOOT_ORDER=0x241
SD_CARD_MAX_RETRIES=3
NET_BOOT_MAX_RETRIES=5
```
In this case I have pointed the TFTP boot at the Synology DHCP server, and set a boot order that will try to boot from a microSD card first, then the network then finally any attached USB storage.
Then we can write a new eeprom binary:
```
rpi-eeprom-config --out pieeprom-new.bin --config bootconf.txt pieeprom.bin
```
and write it in to the eeprom:
```
sudo rpi-eeprom-update -d -f ./pieeprom-new.bin
```
Reboot, leaving the microSD card in for now:
```
sudo reboot
```
and check the new config values with:
```
vcgencmd bootloader_config
```

## Final step on the Synology

Almost! There is a bit off oddness that I haven't figured out yet. The Synology needs the boot files in a different directory to configure it for the PXE bootloader than the Pi will use to boot from. I'm probably doing something dumb with my setup for multiple Pis (and this probably isn't necessary with a single Pi), but for now, on the Pi:

```
sudo cp -r /boot/* /nfs/rpi-tftpboot/
```
then in the Synology console enable PXE boot and point it at the bootloader:

![dhcp_pxe_setting]({{ site.url }}/assets/dhcp_pxe_setting.png)

## And PXE boot

Shutdown the Pi and turn off the power, remove the microSD card and boot the Pi. It should now PXE boot. Boot times are fairly similar to booting from the microSD card in the case of a Synology.

## Debugging when things go wrong

There were a couple of key places I checked to help debug.

1. When you try to PXE boot the Pi plug it in to a monitor and some key boot options and messages will be output. This helped me confirm what the eeprom was doing and ensure it was loading the correct settings.
2. The DHCP logs on the Synology also contain useful info. It took me a while to track the right log down. You need to enable SSH on the Synology then ssh in. Check the log at:

```
sudo tail -f /var/log/dnsmasq.log
```

## Performance tests

I did some simple performance tests to assess performance of the following:

1. Sandisk Extreme Pro 64GB microSD card
2. Synology diskstation DS212+ PXE boot, single client
3. Synology diskstation DS720+ PXE boot, single client
4. Synology diskstation DS720+ PXE boot, 3 clients
5. G-Drive mobile USB 3.0 HDD

The DS720+ is one of the newest models, configured with 6GB RAM, Seagate Ironwolf 7200rpm disks and Seagate Ironwold NVMe SSD cache, so I was expecting it to be significantly faster than the older DS212+ with 0.5GB RAM, 5400rpm drives and no cache.

Roughly I ran 4 runs of the following commands, discarded the first result and average the last 3 results. Bit different for the read tests as I couldn't find a way to disable the disk cache so did the test with multiple runs after a reboot. Overall the tests are a bit crude, but it gave me the comparisons I wanted rather than absolute results.

### Write testing

Write test with dd command to measure server throughput:
```
dd if=/dev/zero of=/tmp/test1.img bs=1G count=1 oflag=dsync
```
Write test with dd command to measure server latency:
```
dd if=/dev/zero of=/tmp/test2.img bs=512 count=1000 oflag=dsync
```

### Read testing

Read test with dd command to measure server throughput:
```
dd if=/dev/zero of=~/tmp/test1.img bs=1G count=1 oflag=dsync
sudo reboot
time dd if=~/tmp/test1.img of=/dev/null bs=4k
```

### Comparison results

Just to reiterate these are relative comparisons of storage read and write speeds for various Raspberry Pi options. I'm pleased with the multiple client PXE boot compared to microSD.

![RaspberryPi_storage_performance_comparison]({{ site.url }}/assets/RaspberryPi_storage_performance_comparison.png)

A single client PXE booted is faster than the high performance Sandisk card, and multiple clients compare favorably to the microSD card option. Given the test with multiple clients involved simultaneous intensive read/write tests I might expect real world performance to be faster than microSD card. Whilst getting the option of easier backups and automation and better long term reliability.

## Next steps

I'd like to automate the process with Ansible to save time with multiple Pis and increase reliability of the process.

## Resources I used to get this working

### Setup

The following resources were useful, the first one in particular:

* *[https://www.definit.co.uk/2020/03/pxe-booting-raspberry-pi-4-with-synology-and-ubiquiti-edgerouter/](https://www.definit.co.uk/2020/03/pxe-booting-raspberry-pi-4-with-synology-and-ubiquiti-edgerouter/)*
* [https://www.virtuallyghetto.com/2020/07/two-methods-to-network-boot-raspberry-pi-4.html](https://www.virtuallyghetto.com/2020/07/two-methods-to-network-boot-raspberry-pi-4.html)
* [https://blog.ronnyvdb.net/2020/05/10/howto-raspberry-pi-4-pxe-network-boot/](https://blog.ronnyvdb.net/2020/05/10/howto-raspberry-pi-4-pxe-network-boot/)
* [https://www.synology.com/en-global/knowledgebase/DSM/tutorial/Management/How_to_implement_PXE_with_Synology_NAS](https://www.synology.com/en-global/knowledgebase/DSM/tutorial/Management/How_to_implement_PXE_with_Synology_NAS)
* [https://www.synoforum.com/resources/pxe-boot-via-synology-nas.15/](https://www.synoforum.com/resources/pxe-boot-via-synology-nas.15/)

### bootloader config

The raspberry pi bootloader is a key part of PXE booting, the config options are worth understanding.

* [https://www.raspberrypi.org/documentation/hardware/raspberrypi/bcm2711_bootloader_config.md](https://www.raspberrypi.org/documentation/hardware/raspberrypi/bcm2711_bootloader_config.md)
