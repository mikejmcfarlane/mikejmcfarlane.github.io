# Raspberry Pi - USB gadget mode for iPad

- [Raspberry Pi - USB gadget mode for iPad](#raspberry-pi---usb-gadget-mode-for-ipad)
  - [Useful resources](#useful-resources)
  - [Setup](#setup)

## Useful resources

[Connect to Your Raspberry Pi Over USB Using Gadget Mode](https://howchoo.com/pi/raspberry-pi-gadget-mode)

## Setup

1. Create microSD card with Raspbian minimal, don't eject yet
2. In `boot/config.txt`, append to end of file:

```
dtoverlay=dwc2
```

3. In `boot/`:

```
touch ssh
```

4. In `boot/cmdline.txt`, after `rootwait` add:

```
modules-load=dwc2,g_ether
```
e.g.
```
console=serial0,115200 console=tty1 root=PARTUUID=6feefb72-02 rootfstype=ext4 elevator=deadline fsck.repair=yes rootwait modules-load=dwc2,g_ether
```

5. Connect Pi to iPad using USB C on both, or if need to charge iPad, then use Apple USB/HDMI adapter with USB A cable from adapter to USB C on Pi.
6. Run `raspi-config` to set wifi, change hostname, expand filesystem etc
7. `ssh pi@raspberrypi.local`, default password `raspberry`
8. `sudo apt update; sudo apt upgrade`
9. `sudo apt install ansible git vim`