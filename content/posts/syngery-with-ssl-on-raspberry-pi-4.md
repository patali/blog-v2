+++
date = 2020-02-22T06:30:00Z
disable_comments = false
tags = ["workspace", "raspberry pi", "synergy ssl", "symless", "synergy"]
title = "Synergy with SSL on Raspberry Pi 4"
type = ""

+++
End of 2019 my daily driver 2015 MacBook Pro was beginning to show signs of ageing. Especially when I’m running multiple IDE’s, terminal emulators and browsers running in the background. Also two monitors (Laptop’s built in and an external 1080p monitor) was just not enough. I had to constantly switch between IDE and system monitoring and IRC/Slack clients. But driving more than two monitors would be next to impossible on it. However the processor inside is pretty decent for my current requirement of Android and web development. To solve this dilemma of to upgrade or not upgrade I decided to make a pros and cons list

**Upgrade**

* Pros:
  * Shiny new computer
  * Snappier context switching
  * Faster compilation times
* Cons:
  * Expensive!
  * Wasteful as older hardware is still decent

**!Upgrade**

* Pros:
  * No (or minimal) cost to company :P
  * Might learn something new in the process
  * Recycling old hardware and prolonging the use of existing hardware
* Cons:
  * Less screen real estate
  * Higher context switching times
  * No RAM or Speed upgrade

It’s pretty obvious that I had to go try the _learn/reuse_ route. I really didn’t want to get a new machine unless I’m compelled to do so. I started looking around for some ideas on how to offload some of my windows onto a different monitor. All my comms windows on one monitor or side-by-side on two monitors, Web browser/reference windows on another and finally IDE on one full monitor. Distraction free.

After poking around various corners of the internet, I decided to go with the new Raspberry Pi 4 with 4gb RAM driving an old monitor that was lying around the office. Ubuntu 19.10 arm64 was recently released for RPI 4, perfect timing. I need a proper terminal emulator with font and color support so I use a combination of OpenBox + Awesome tiling window manager. The combination is pretty light weight on RAM and CPU usage. Finally to glue it all together I use Synergy to share my Mac’s keyboard and mouse with the RPi.

### Running Synergy on Ubuntu 19.10 arm64

My Mac is the Synergy server as I want to share all input devices connected to it with all connected clients (RPi4 in this case). So I first installed the latest Ubuntu 19.10 release. Synergy client is available on apt-get but it’s several versions behind. Fortunately [Symless ](https://symless.com/synergy)provides an installer for Raspbian. You get access to it once you purchase a license from them and login.

Since I need to relay keyboard inputs to login screen in RPI, I have to manually make some changes to auto start Synergy before the login screen. I use LightDM + [SlickGreeter](https://github.com/linuxmint/slick-greeter), so I followed [these](https://help.ubuntu.com/community/SynergyHowto#Autostart_Synergy_before_logging_in_.28LightDM.29) steps to auto start synergy client.

One **important** piece of information that I could not find online is that if you using SSL with Synergy, you will have copy over the Synergy config (found here: \~/.synergy) into root user config folder as well. Basically the certificate signature needs to be accessible by the root user, which is stored in the Synergy config file. Alternatively you can login as a root user and run Synergy client to set things up.

Finally after a lot of tinkering this is what my current setup looks like.

* _Top Left Monitor_: Mac driven. Runs Browser + Slack and other reference/reading windows in virtual desktops
* _Right Monitor_: RPI driven. Runs IRC + Terminal. Runs additional terminals on other virtual desktops
* _Mac display_: Primary IDE. Mail client on other virtual desktops.

![](https://patali-blogs-bucket.s3.us-east-2.amazonaws.com/patali.in/workspace-pataliin.jpg)