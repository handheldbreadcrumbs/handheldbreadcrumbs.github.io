---
layout: post
title:  "Building Palm OS Emulator from sources"
date:   2017-05-11 17:10:56
categories: build pose sources
---

For some time I have an interest in [Palm PDA](https://en.wikipedia.org/wiki/Palm_(PDA)){:target="_blank"} devices, since I really enjoy finding information about different kinds of computing devices in general, mostly microcomputers from 1970s, 1980s I find personal digital assistants very interesting too. I am not quite sure whether Palm devices can be called retro computers yet, but it is my sort of version of [retrocomputing](https://en.wikipedia.org/wiki/Retrocomputing){:target="_blank"}. As with other computing devices an integral part of Palm PDA is its software. Normally software will be running on real hardware, but there are other cases when software can be running in emulated environment for example when writing, testing or debugging software. In this post I am going to document steps I did to build emulator software - Palm OS Emulator from source code available at [https://sourceforge.net/projects/pose/](https://sourceforge.net/projects/pose/){:target="_blank"} in GNU/Linux environment i.e. Debian operating system.

Software building process will happen inside chroot environment created with [Debootstrap](https://wiki.debian.org/Debootstrap){:target="_blank"}. It is a tool which will install a Debian base system into a subdirectory of another, already installed system. 

First of all we need to install debootstrap if not done already. On Debian or Ubuntu it can be done just by running this command from terminal {% highlight bash %}
sudo apt-get install debootstrap
{% endhighlight %}

