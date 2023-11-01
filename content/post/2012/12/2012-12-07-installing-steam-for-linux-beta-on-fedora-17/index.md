---
title: "Installing Steam for Linux Beta on Fedora 17"
date: "2012-12-07"
categories: 
  - "fedora"
---

**UPDATE:** _Since the original post, the download of Team Fortress 2 completed and I hit a problem. The post has been amended with the solution._

It sounds like a lot more people got access to the Steam for Linux beta yesterday, including me. An announcement on [steamcommunity.com](http://steamcommunity.com/games/221410/announcements/detail/1749910796493493022) says:

> We've just expanded the limited public beta by a large amount - which means another round of email notifications - so check your inbox!

The official download is a deb package for Ubuntu. My laptop runs Fedora 17. I was pleasantly surprised to see that there was an unofficial Fedora repository already ready to use. Here is how I installed the beta on my laptop running Fedora 17:

`$ wget http://spot.fedorapeople.org/steam/steam.repo ... $ sudo mv steam.repo /etc/yum.repos.d/ $ sudo yum install steam ... $ rpm -q steam steam-1.0.0.14-3.fc17.i686`

Once installation was completed, I ran the steam client from the same terminal:

`$ steam`

The first time I ran the steam client it automatically created `/home/rbryant/Steam` and downloaded about 100 kB of updates. Once the updates completed, the login screen came up. I closed the steam client and ran it again. I got a warning dialog that said:

> Unable to copy /home/rbryant/Steam/bin\_steam.sh to /usr/bin/steam, please contact your system administrator.

This was a bit odd since the app that I had been running was already `/usr/bin/steam`. I suspect this is just automatically installing a new version based on what was downloaded with the updates. Based on the output in my terminal, I can see that before this warning came up, steam tried to find `gksudo`, `kdesudo`, or `xterm` and then gave up. I went ahead and installed xterm.

`$ sudo yum install xterm`

When running steam yet again, it popped up an xterm window to ask me to type in my password. This only happened once. Subsequent runs of the steam client in my terminal went straight to the login window.

From there I finally decided to log in using my existing steam account. I confirmed access to my account on a new computer and was in. I kicked off a download of Team Fortress 2 Beta for Linux.

Once the game download was complete, I clicked Play. The first time I tried, it failed with the following error:

> Required OpenGL extension "GL\_EXT\_texture\_compression\_s3tc" is not supported. Please install S3TC texture support.

To fix this, I had to add the rpmfusion repositories to my machine and install the `libtxc_dxtn` package.

`$ sudo yum localinstall --nogpgcheck http://download1.rpmfusion.org/free/fedora/rpmfusion-free-release-stable.noarch.rpm http://download1.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-stable.noarch.rpm

$ sudo yum install libtxc_dxtn`

If you're running 64-bit Linux, you will actually need the 32-bit version of this library to fix the game. Install it by running:

`$ sudo yum install libtxc_dxtn.i686`

Once all of that was done, the game launched successfully and I was able to start a training session.

Enjoy!
