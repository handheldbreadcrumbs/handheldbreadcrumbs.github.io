---
layout: post
title:  "HotSync pda device through IrDA on linux"
date:   2016-02-11 22:09:56
categories: hotsync irda pda handspring
---

It's a year 2016 and I am attempting to synchronize PDA from early two thousands with my pc. This is more or less a memo for myself. I am sitting on Debian flavor of linux and so far I have read a bunch or articles on topic that I found googling the internet. It is fortunate or not so fortunate for me, nowadays it is easier to setup IrDA connectivity and most of the things just work right off the bat without much struggle.  
I did all this experimenting on freshly installed system and just before started to setup basic configuration I checked few things. In short one needs to install irda-utils,pilot-link packages as root, add your ordinary user to dialout group and then reboot. After reboot plugin IrDA to USB dongle and check dmesg for any issues that could appear and in case of no problems find out with what device name dongle was registered. Again as root load such kernel modules exactly in this order irda, ircomm, ircomm-tty and execute irattach <device name> -s. In this case device name was irda0.
{% highlight bash %}
wolf@chimp:~$ 
wolf@chimp:~$ uname -a
Linux chimp 3.16.0-4-amd64 #1 SMP Debian 3.16.7-ckt20-1+deb8u3 (2016-01-17) x86_64 GNU/Linux
wolf@chimp:~$ 
{% endhighlight %}
I was curious what could be seen in the kernel ring buffer before I plugged in IrDA to USB dongle.
{% highlight bash %}
wolf@chimp:~$ 
wolf@chimp:~$ dmesg | less
wolf@chimp:~$ dmesg | tail
[   13.795282] cfg80211:   (2402000 KHz - 2472000 KHz @ 40000 KHz), (N/A, 2000 mBm), (N/A)
[   13.795285] cfg80211:   (2457000 KHz - 2482000 KHz @ 40000 KHz), (N/A, 2000 mBm), (N/A)
[   13.795287] cfg80211:   (2474000 KHz - 2494000 KHz @ 20000 KHz), (N/A, 2000 mBm), (N/A)
[   13.795290] cfg80211:   (5170000 KHz - 5250000 KHz @ 80000 KHz, 160000 KHz AUTO), (N/A, 2000 mBm), (N/A)
[   13.795293] cfg80211:   (5250000 KHz - 5330000 KHz @ 80000 KHz, 160000 KHz AUTO), (N/A, 2000 mBm), (0 s)
[   13.795295] cfg80211:   (5490000 KHz - 5730000 KHz @ 160000 KHz), (N/A, 2000 mBm), (0 s)
[   13.795298] cfg80211:   (5735000 KHz - 5835000 KHz @ 80000 KHz), (N/A, 2000 mBm), (N/A)
[   13.795300] cfg80211:   (57240000 KHz - 63720000 KHz @ 2160000 KHz), (N/A, 0 mBm), (N/A)
[   14.122903] atl1c 0000:05:00.0: irq 54 for MSI/MSI-X
[   14.123421] atl1c 0000:05:00.0: atl1c: eth0 NIC Link is Up<100 Mbps Full Duplex>
wolf@chimp:~$
{% endhighlight %}
and after plugging it in
{% highlight bash %}
wolf@chimp:~$ dmesg | tail
[   14.122903] atl1c 0000:05:00.0: irq 54 for MSI/MSI-X
[   14.123421] atl1c 0000:05:00.0: atl1c: eth0 NIC Link is Up<100 Mbps Full Duplex>
[  117.236369] usb 4-1.6: USB disconnect, device number 4
[  120.765623] usb 4-1.6: new full-speed USB device number 5 using ehci-pci
[  120.858870] usb 4-1.6: New USB device found, idVendor=066f, idProduct=4200
[  120.858874] usb 4-1.6: New USB device strings: Mfr=1, Product=2, SerialNumber=0
[  120.858877] usb 4-1.6: Product:  IrDA/USB Bridge
[  120.858879] usb 4-1.6: Manufacturer:  Sigmatel Inc 
[  120.859679] SigmaTel STIr4200 IRDA/USB found at address 5, Vendor: 66f, Product: 4200
[  120.859932] stir4200 4-1.6:1.0: IrDA: Registered SigmaTel device irda0
wolf@chimp:~$
{% endhighlight %}
Obviously dongle was recognized and it seems that it was registered without any issues, at least at this point I was not able to see any errors. I proceeded with some more checks on currently loaded kernel modules related to usb functionality and checking if there are proper devices and what kind of access rights do they have.
{% highlight bash %}
wolf@chimp:~$ lsmod | grep usb
usbhid                 44460  0 
hid                   102264  2 hid_generic,usbhid
usb_storage            56215  0 
scsi_mod              191405  5 sg,usb_storage,libata,sd_mod,sr_mod
usbcore               195427  6 usb_storage,ehci_hcd,ehci_pci,usbhid,stir4200,xhci_hcd
usb_common             12440  1 usbcore
wolf@chimp:~$ ls -l /dev/ttyS?
crw-rw---- 1 root dialout 4, 64 Feb  6 12:22 /dev/ttyS0
crw-rw---- 1 root dialout 4, 65 Feb  6 12:22 /dev/ttyS1
crw-rw---- 1 root dialout 4, 66 Feb  6 12:22 /dev/ttyS2
crw-rw---- 1 root dialout 4, 67 Feb  6 12:22 /dev/ttyS3
wolf@chimp:~$ 
wolf@chimp:~$ ls -l /dev/ircomm?
ls: cannot access /dev/ircomm?: No such file or directory
wolf@chimp:~$ 
{% endhighlight %}

Then in new terminal window I changed my current user to superuser i.e. root and installed few packages that are needed for work with IrDA
{% highlight bash %}
root@chimp:/etc/apt# 
root@chimp:/etc/apt# apt-get install irda-utils pilot-link
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following extra packages will be installed:
  libopenobex1 libpisock9 openobex-apps setserial
Suggested packages:
  libgsmme1c102 liblinc1 obexftp jpilot kpilot gnome-pilot evolution claws-mail sylpheed
The following NEW packages will be installed:
  irda-utils libopenobex1 libpisock9 openobex-apps pilot-link setserial
0 upgraded, 6 newly installed, 0 to remove and 0 not upgraded.
Need to get 905 kB of archives.
After this operation, 2,125 kB of additional disk space will be used.
Do you want to continue? [Y/n] Y
Get:1 http://debian.koyanet.lv/debian/ jessie/main irda-utils amd64 0.9.18-12 [98.1 kB]
Get:2 http://debian.koyanet.lv/debian/ jessie/main libopenobex1 amd64 1.5-2.1 [24.5 kB]
Get:3 http://debian.koyanet.lv/debian/ jessie/main libpisock9 amd64 0.12.5-dfsg-1 [272 kB]
Get:4 http://debian.koyanet.lv/debian/ jessie/main openobex-apps amd64 1.5-2.1 [39.5 kB]
Get:5 http://debian.koyanet.lv/debian/ jessie/main pilot-link amd64 0.12.5-dfsg-1 [419 kB]
Get:6 http://debian.koyanet.lv/debian/ jessie/main setserial amd64 2.17-48 [51.3 kB]
Fetched 905 kB in 0s (3,067 kB/s)   
Preconfiguring packages ...
Selecting previously unselected package irda-utils.
(Reading database ... 75770 files and directories currently installed.)
Preparing to unpack .../irda-utils_0.9.18-12_amd64.deb ...
Unpacking irda-utils (0.9.18-12) ...
Selecting previously unselected package libopenobex1.
Preparing to unpack .../libopenobex1_1.5-2.1_amd64.deb ...
Unpacking libopenobex1 (1.5-2.1) ...
Selecting previously unselected package libpisock9.
Preparing to unpack .../libpisock9_0.12.5-dfsg-1_amd64.deb ...
Unpacking libpisock9 (0.12.5-dfsg-1) ...
Selecting previously unselected package openobex-apps.
Preparing to unpack .../openobex-apps_1.5-2.1_amd64.deb ...
Unpacking openobex-apps (1.5-2.1) ...
Selecting previously unselected package pilot-link.
Preparing to unpack .../pilot-link_0.12.5-dfsg-1_amd64.deb ...
Unpacking pilot-link (0.12.5-dfsg-1) ...
Selecting previously unselected package setserial.
Preparing to unpack .../setserial_2.17-48_amd64.deb ...
Unpacking setserial (2.17-48) ...
Processing triggers for systemd (215-17+deb8u3) ...
Processing triggers for man-db (2.7.0.2-5) ...
Setting up irda-utils (0.9.18-12) ...
/var/lib/dpkg/info/irda-utils.postinst: line 141: /dev/MAKEDEV: No such file or directory
Setting up libopenobex1 (1.5-2.1) ...
Setting up libpisock9 (0.12.5-dfsg-1) ...
Setting up openobex-apps (1.5-2.1) ...
Setting up pilot-link (0.12.5-dfsg-1) ...
Setting up setserial (2.17-48) ...
removing the old setserial entry in the rcn.d directories
Update complete.
update-rc.d: warning: start and stop actions are no longer supported; falling back to defaults
update-rc.d: warning: start and stop actions are no longer supported; falling back to defaults
update-rc.d: warning: start and stop actions are no longer supported; falling back to defaults
update-rc.d: warning: start and stop actions are no longer supported; falling back to defaults
Saving state of known serial devices... backing up /var/lib/setserial/autoserial.conf done.
Processing triggers for systemd (215-17+deb8u3) ...
Processing triggers for libc-bin (2.19-18+deb8u2) ...
root@chimp:/etc/apt# 
{% endhighlight %}
additionally for convenience not to change access rights for devices all the time I added myself to dialout group
{% highlight bash %}
root@chimp:/etc/apt# 
root@chimp:/etc/apt# gpasswd -a wolf dialout
Adding user wolf to group dialout
root@chimp:/etc/apt# 
{% endhighlight %}
Time for reboot and after login let's check some basic functionality. After reboot, plugging in IrDA-USB dongle, checking dmesg for errors.
{% highlight bash %}
wolf@chimp:~$ 
wolf@chimp:~$ dmesg | tail
[   14.175232] atl1c 0000:05:00.0: atl1c: eth0 NIC Link is Up<100 Mbps Full Duplex>
[  284.941500] usb 4-1.6: new full-speed USB device number 4 using ehci-pci
[  285.034694] usb 4-1.6: New USB device found, idVendor=066f, idProduct=4200
[  285.034699] usb 4-1.6: New USB device strings: Mfr=1, Product=2, SerialNumber=0
[  285.034701] usb 4-1.6: Product:  IrDA/USB Bridge
[  285.034703] usb 4-1.6: Manufacturer:  Sigmatel Inc 
[  285.138477] NET: Registered protocol family 23
[  285.157333] SigmaTel STIr4200 IRDA/USB found at address 4, Vendor: 66f, Product: 4200
[  285.157682] stir4200 4-1.6:1.0: IrDA: Registered SigmaTel device irda0
[  285.157718] usbcore: registered new interface driver stir4200
wolf@chimp:~$ 
{% endhighlight %}
rechecking access rights
{% highlight bash %}
wolf@chimp:~$ 
wolf@chimp:~$ ls -l /dev/ttyS?
crw-rw---- 1 root dialout 4, 64 Feb  6 12:37 /dev/ttyS0
crw-rw---- 1 root dialout 4, 65 Feb  6 12:37 /dev/ttyS1
crw-rw---- 1 root dialout 4, 66 Feb  6 12:37 /dev/ttyS2
crw-rw---- 1 root dialout 4, 67 Feb  6 12:37 /dev/ttyS3
wolf@chimp:~$ ls -l /dev/ircomm?
ls: cannot access /dev/ircomm?: No such file or directory
wolf@chimp:~$ 
{% endhighlight %}
{% highlight bash %}
root@chimp:~# 
root@chimp:~# modprobe irda
{% endhighlight %}
checking dmesg
{% highlight bash %}
wolf@chimp:~$ 
wolf@chimp:~$ dmesg | tail
[  285.034699] usb 4-1.6: New USB device strings: Mfr=1, Product=2, SerialNumber=0
[  285.034701] usb 4-1.6: Product:  IrDA/USB Bridge
[  285.034703] usb 4-1.6: Manufacturer:  Sigmatel Inc 
[  285.138477] NET: Registered protocol family 23
[  285.157333] SigmaTel STIr4200 IRDA/USB found at address 4, Vendor: 66f, Product: 4200
[  285.157682] stir4200 4-1.6:1.0: IrDA: Registered SigmaTel device irda0
[  285.157718] usbcore: registered new interface driver stir4200
[  399.834579] net irda0: usb receive submit error: -1
[  399.934620] net irda0: usb receive submit error: -1
[  826.967831] net irda0: usb receive submit error: -1
wolf@chimp:~$ 
root@chimp:~# modprobe ircomm
root@chimp:~# 
{% endhighlight %}
checking dmesg
{% highlight bash %}
wolf@chimp:~$ dmesg | tail
[  285.034701] usb 4-1.6: Product:  IrDA/USB Bridge
[  285.034703] usb 4-1.6: Manufacturer:  Sigmatel Inc 
[  285.138477] NET: Registered protocol family 23
[  285.157333] SigmaTel STIr4200 IRDA/USB found at address 4, Vendor: 66f, Product: 4200
[  285.157682] stir4200 4-1.6:1.0: IrDA: Registered SigmaTel device irda0
[  285.157718] usbcore: registered new interface driver stir4200
[  399.834579] net irda0: usb receive submit error: -1
[  399.934620] net irda0: usb receive submit error: -1
[  826.967831] net irda0: usb receive submit error: -1
[  907.882011] IrCOMM protocol (Dag Brattli)
wolf@chimp:~$ 
{% endhighlight %}
{% highlight bash %}
wolf@chimp:~$ 
wolf@chimp:~$ ls -l /dev/ttyS?
crw-rw---- 1 root dialout 4, 64 Feb  6 12:37 /dev/ttyS0
crw-rw---- 1 root dialout 4, 65 Feb  6 12:37 /dev/ttyS1
crw-rw---- 1 root dialout 4, 66 Feb  6 12:37 /dev/ttyS2
crw-rw---- 1 root dialout 4, 67 Feb  6 12:37 /dev/ttyS3
wolf@chimp:~$ ls -l /dev/ircomm?
ls: cannot access /dev/ircomm?: No such file or directory
wolf@chimp:~$ 
root@chimp:~# modprobe ircomm-tty
root@chimp:~# 
{% endhighlight %}
{% highlight bash %}
wolf@chimp:~$ dmesg | tail
[  285.138477] NET: Registered protocol family 23
[  285.157333] SigmaTel STIr4200 IRDA/USB found at address 4, Vendor: 66f, Product: 4200
[  285.157682] stir4200 4-1.6:1.0: IrDA: Registered SigmaTel device irda0
[  285.157718] usbcore: registered new interface driver stir4200
[  399.834579] net irda0: usb receive submit error: -1
[  399.934620] net irda0: usb receive submit error: -1
[  826.967831] net irda0: usb receive submit error: -1
[  907.882011] IrCOMM protocol (Dag Brattli)
[ 1013.850132] net irda0: usb receive submit error: -1
[ 1013.950179] net irda0: usb receive submit error: -1
{% endhighlight %}
{% highlight bash %}
wolf@chimp:~$ ls -l /dev/ttyS?
crw-rw---- 1 root dialout 4, 64 Feb  6 12:37 /dev/ttyS0
crw-rw---- 1 root dialout 4, 65 Feb  6 12:37 /dev/ttyS1
crw-rw---- 1 root dialout 4, 66 Feb  6 12:37 /dev/ttyS2
crw-rw---- 1 root dialout 4, 67 Feb  6 12:37 /dev/ttyS3
wolf@chimp:~$ 
wolf@chimp:~$ ls -l /dev/ircomm?
crw-rw---- 1 root dialout 161, 0 Feb  6 12:54 /dev/ircomm0
crw-rw---- 1 root dialout 161, 1 Feb  6 12:54 /dev/ircomm1
crw-rw---- 1 root dialout 161, 2 Feb  6 12:54 /dev/ircomm2
crw-rw---- 1 root dialout 161, 3 Feb  6 12:54 /dev/ircomm3
crw-rw---- 1 root dialout 161, 4 Feb  6 12:54 /dev/ircomm4
crw-rw---- 1 root dialout 161, 5 Feb  6 12:54 /dev/ircomm5
crw-rw---- 1 root dialout 161, 6 Feb  6 12:54 /dev/ircomm6
crw-rw---- 1 root dialout 161, 7 Feb  6 12:54 /dev/ircomm7
crw-rw---- 1 root dialout 161, 8 Feb  6 12:54 /dev/ircomm8
crw-rw---- 1 root dialout 161, 9 Feb  6 12:54 /dev/ircomm9
wolf@chimp:~$
{% endhighlight %}
{% highlight bash %}
root@chimp:~# 
root@chimp:~# irattach irda0 -s
root@chimp:~# 
wolf@chimp:~$ dmesg | tail
[  285.138477] NET: Registered protocol family 23
[  285.157333] SigmaTel STIr4200 IRDA/USB found at address 4, Vendor: 66f, Product: 4200
[  285.157682] stir4200 4-1.6:1.0: IrDA: Registered SigmaTel device irda0
[  285.157718] usbcore: registered new interface driver stir4200
[  399.834579] net irda0: usb receive submit error: -1
[  399.934620] net irda0: usb receive submit error: -1
[  826.967831] net irda0: usb receive submit error: -1
[  907.882011] IrCOMM protocol (Dag Brattli)
[ 1013.850132] net irda0: usb receive submit error: -1
[ 1013.950179] net irda0: usb receive submit error: -1
wolf@chimp:~$ 
{% endhighlight %}
doing some basic command to understand if this actually works...
{% highlight bash %}
wolf@chimp:~$ 
wolf@chimp:~$ pilot-xfer -p /dev/ircomm0 -l 

   Listening for incoming connection on /dev/ircomm0... connected!

   Reading list of databases in RAM...
   SF-EE_SysHeap
   SFT-EE_SYMSFTES
   SFT-EE_SYMSFPOR
   SFT-EE_SYMSFPAR
   SFT-EE_SYMSFHS
   SFT-EE_SYMSFDIA
   SFT-EE_SYMSFBDR
   SFT-EE_SYMSF2C
   AddressDB
   DatebookDB
   MailDB
   MemoDB
   ConnectionDB
   NetworkDB
   ToDoDB
   SFE_DiagUtil
   SF-EE_SYMBOLDIAG
   Graffiti
   psysLaunchDB
   Graffiti ShortCuts
   Unsaved Preferences
   Net Prefs
   System MIDI Sounds
   Saved Preferences

   List complete. 24 files found.


   Thank you for using pilot-link.

wolf@chimp:~$ 
{% endhighlight %}
I think that this setup was successful.
