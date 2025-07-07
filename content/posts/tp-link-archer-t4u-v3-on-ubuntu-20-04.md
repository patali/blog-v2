+++
date = 2020-04-26T06:30:00Z
disable_comments = false
tags = ["engineering", "ubuntu 20.04", "kernel", "MOK", "secure boot", "open source", "drivers", "kernel-modules"]
title = "TP-Link Archer T4U v3 on Ubuntu 20.04"
type = ""

+++
To help with my work I recently installed Ubuntu 20.04 on my home computer. 20.04 is surprisingly snappy, crisp and stable on my home computer (Ryzen 7 1700, 32GB 3200Mhz DDR4, GTX 1080, NVME drives). However I hit upon an interesting problem with a USB WiFi dongle that I use TP-LINK Archer T4U v3, It did not work out of the box. So as expected I had to look at the self compiling of drivers route.

I had to go through a bunch of things to get it to work involving Secure Boot, self signed Kernel modules etc. So I thought it might be best to document it here as I'm sure to forget it eventually, also hopefully it might be helpful to someone.

Here's what I'm going to write about

1. Finding the right driver source and compiling
2. Generating custom Secure Boot keys
3. Signing the compiled kernel module files
4. Loading the kernel module at boot time

## Driver source and compilation

TP-Link has driver source code available for download on their support website. It was last updated sometime in 2018. So naturally it wouldn't compile without some code changes. Thankfully a bunch of good samaritans has made the modifications necessary and uploaded to different GitHub repositories.

After trying a few of them I settled with [borting/rtw88-usb](https://github.com/borting/rtw88-usb)
The compile instructions were as simple as running make. If you are reading the documentation in the repository then I advise you to not run the install instructions just yet. We have to sign the kernel module first.

## Secure Boot and self signing

If you have Secure Boot enabled and if you are loading custom Kernel modules at boot time then they have to be signed or else they will be ignored by the kernel (As to why this is the case it is a very interesting subject to [read](https://wiki.ubuntu.com/UEFI/SecureBoot) on).

To sign the above generated modules we will have to first generate a key and enroll it to Secure Boot keys list.

First generate the key

```bash
openssl req -new -x509 -newkey rsa:2048 -keyout MyMOK.priv -outform DER -out MyMOK.der -nodes -days 36500 -subj "/CN=Key for signing custom kernel modules/"
```

Register the key with Secure Boot

```bash
sudo mokutil --import MyMOK.der
```

Reboot the system. Secure Boot should now ask you some questions here regarding MOK (Machine Owner Key) enrollment. Follow the detailed instructions mentioned here: [https://sourceware.org/systemtap/wiki/SecureBoot](https://sourceware.org/systemtap/wiki/SecureBoot)

Once it reboots again and comes back to Ubuntu, verify if the key was enrolled properly

```bash
mokutil --test-key MyMOK.der
```

## Signing the custom modules with MOK

There were two .ko modules ( **rtw88.ko & rtwusb.ko** ) generated in that repository after running `make`

Sign the modules by running the following commands in a terminal

```bash
sudo /usr/src/linux-headers-$(uname -r)/scripts/sign-file sha256 ./MyMOK.priv ./MyMOK.der rtw88.ko
```

```bash
sudo /usr/src/linux-headers-$(uname -r)/scripts/sign-file sha256 ./MyMOK.priv ./MyMOK.der rtwusb.ko
```

Make sure you use the correct path to the signing certificates. Also note that if you rerun `make` you will have to run the signing commands again.

## Loading the drivers on boot time

First move the firmware files to correct directory

```bash
sudo mkdir -p /lib/firmware/rtw88
sudo cp fw/rtw8822* /lib/firmware/rtw88/
```

Move the signed modules to the correct directory

```bash
sudo mkdir -p /lib/modules/`uname -r`/kernel/drivers/net/wireless/realtek/rtw88
sudo cp rtw88.ko /lib/modules/`uname -r`/kernel/drivers/net/wireless/realtek/rtw88
sudo cp rtwusb.ko /lib/modules/`uname -r`/kernel/drivers/net/wireless/realtek/rtw88
```

Run depmod to probe all modules

```bash
sudo depmod -a
```

Next we will create a conf file to inform the system to load our custom kernel modules on boot

```bash
sudo touch /etc/modules-load.d/rtwusb.conf
```

Open the created file in the editor of your choice and add these two lines (separate lines) and save it.

    rtw88
    rtwusb

Restart kernel module loading service

```bash
sudo systemctl start systemd-modules-load
```

The WiFi networking should now appear in your dock. If it doesn't give your system a reboot. You should be able to scan and register yourself to a network.

## Closing notes

This was very interesting rabbit hole to follow, learnt a-bit about Secure Boot, self signing kernel modules. I'm going to list all my references at the end of this post.

The situation around the world is pretty grim right now, please stay home and stay safe, take care of your loved ones. We may very well be in the [darkest timeline](https://community-sitcom.fandom.com/wiki/Darkest_Timeline) as prophesied by Abed & Troy.

### References

* [https://github.com/borting/rtw88-usb](https://github.com/borting/rtw88-usb)
* [https://github.com/Red-Eyed/TP-LINK_Archer_T4U_v3](https://github.com/Red-Eyed/TP-LINK_Archer_T4U_v3)
* [https://wiki.ubuntu.com/UEFI/SecureBoot](https://wiki.ubuntu.com/UEFI/SecureBoot)
* [https://askubuntu.com/questions/760671/could-not-load-vboxdrv-after-upgrade-to-ubuntu-16-04-and-i-want-to-keep-secur/768310#768310](https://askubuntu.com/questions/760671/could-not-load-vboxdrv-after-upgrade-to-ubuntu-16-04-and-i-want-to-keep-secur/768310#768310)
* [https://sourceware.org/systemtap/wiki/SecureBoot](https://sourceware.org/systemtap/wiki/SecureBoot)