---
layout: post
title: PXE boot a raspberry pi 4 from a Synology Diskstation
date: 2020-09-12
---

- [Introduction](#introduction)
- [Setting up the Synology](#setting-up-the-synology)
  - [Setup manual IP](#setup-manual-ip)
- [Resources I used to get this working](#resources-i-used-to-get-this-working)
  - [Setup](#setup)
  - [bootloader config](#bootloader-config)

## Introduction

PXE booting allow a server, in this case a Raspberry Pi 4, to boot from a remote computer, in this case a Synology Diskstation DS720+.

## Setting up the Synology

The first step is to configure the Synology to act as a DHCP server and to serve up the the Raspberry Pi operating system files via [NFS](https://www.raspberrypi.org/documentation/configuration/nfs.md).

### Setup manual IP

![Control Panel]({{ site.url }}/assets/ControlPanel.png)

```bash
sudo mount / /
ls -la
pwd
```

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
