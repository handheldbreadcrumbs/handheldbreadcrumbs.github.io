---
layout: post
title:  "POSE from sources Pt. 2"
date:   2017-06-07 17:10:56
categories: build pose sources
---

This is second post in the series of articles about Palm OS Emulator. You can find first part [here]({{ site.baseurl }}{% post_url 2017-05-11-build-pose %}){:target="_blank"}.  
First of all we need to install debootstrap if not done already. On Debian or Ubuntu it can be done just by running this command from terminal {% highlight bash %}
sudo apt-get install debootstrap
{% endhighlight %}
Before proceeding to install anything let's set variable for absolute path to chroot environment where all files will be stored {% highlight bash %}
export MY_CHROOT=/home/$USER/debian.4.0-etch
{% endhighlight %}
let's create a directory where to store all environment files
{% highlight bash %}
mkdir $MY_CHROOT
{% endhighlight %}
and install Debian base system
{% highlight bash %}
sudo debootstrap --variant=minbase --keyring=/usr/share/keyrings/debian-archive-removed-keys.gpg --verbose --arch i386 etch $MY_CHROOT http://archive.debian.org/debian/
{% endhighlight %}
Complete install log can be seen in this [gist](https://gist.github.com/handheldbreadcrumbs/af702fabc6f0c4407b059648faa2e5ee#file-debian-base-install-log){:target="_blank"}. After successful installation everything should be in debian.4.0-etch directory under home folder of current user, then I am running a command with a specified root directory. Specified root directory is $MY_CHROOT and command is not specified explicitly and it defaults to $SHELL, in my case it is /bin/bash, that is invoked with the -i option.
 
In short I am running this command below
{% highlight bash %}
sudo chroot $MY_CHROOT
{% endhighlight %}
after than I am going to install this list of packages
{% highlight bash %}
# Compiler drivers for the GNU Compiler Collection 
gcc-2.95 
g++-2.95 
# utility to list, test and extract compressed files in a ZIP archive
unzip
build-essential #  
xorg-dev
    
libtool
# tool used to generate configuration scripts
autoconf
# tool used to generate the Makefile.in file from programmer-defined file Makefile.am
automake
# a filter for paging through text one screenful at a time
less
# Vi IMproved, a programmers text editor
vim
{% endhighlight %}
{% highlight bash %}
apt-get update && apt-get install -y gcc-2.95 g++-2.95 unzip build-essential xorg-dev libtool autoconf automake less vim
{% endhighlight %}
then I need to understand where are gcc and g++ located
{% highlight bash %}
chswj:~#
chswj:~# command -v g{cc,++}
/usr/bin/gcc
/usr/bin/g++
{% endhighlight %}
let's change shell working directory to /usr/bin where gcc, g++ are and check their properties with ls command
{% highlight bash %}
chswj:~# cd /usr/bin
chswj:/usr/bin#
chswj:/usr/bin# ls -l g{cc,++}
lrwxrwxrwx 1 root root 7 May 26 18:07 g++ -> g++-4.1
lrwxrwxrwx 1 root root 7 May 26 18:07 gcc -> gcc-4.1
chswj:/usr/bin#
{% endhighlight %}
gcc and g++ are symbolic links to specific version of both drivers. Let's check what else we have there related to these compiler drivers.
{% highlight bash %}
chswj:/usr/bin#
chswj:/usr/bin# ls -l | grep -E "gcc|g\+\+"
-rwxr-xr-x  1 root root       428 May  7  2006 c89-gcc
-rwxr-xr-x  1 root root       451 May  7  2006 c99-gcc
lrwxrwxrwx  1 root root         7 May 26 18:07 g++ -> g++-4.1
-rwxr-xr-x  1 root root     74244 Jul 13  2006 g++-2.95
-rwxr-xr-x  1 root root    183444 Dec 10  2006 g++-4.1
lrwxrwxrwx  1 root root         7 May 26 18:07 gcc -> gcc-4.1
-rwxr-xr-x  1 root root     74104 Jul 13  2006 gcc-2.95
-rwxr-xr-x  1 root root    183444 Dec 10  2006 gcc-4.1
lrwxrwxrwx  1 root root        10 May 26 18:07 gccbug -> gccbug-4.1
-rwxr-xr-x  1 root root     16283 Dec 10  2006 gccbug-4.1
lrwxrwxrwx  1 root root         7 May 26 18:07 i486-linux-gnu-g++ -> g++-4.1
lrwxrwxrwx  1 root root         7 May 26 18:07 i486-linux-gnu-g++-4.1 -> g++-4.1
lrwxrwxrwx  1 root root         7 May 26 18:07 i486-linux-gnu-gcc -> gcc-4.1
lrwxrwxrwx  1 root root         7 May 26 18:07 i486-linux-gnu-gcc-4.1 -> gcc-4.1
chswj:/usr/bin#
{% endhighlight %}
From what I have learned, I need to relink these symlinks to version 2.95, I am going to remove gcc and g++ and make symbolic links to gcc-2.95, g++-2.95 instead.
{% highlight bash %}
chswj:/usr/bin#
chswj:/usr/bin# rm gcc
chswj:/usr/bin# ln -s gcc-2.95 gcc
chswj:/usr/bin# rm g++
chswj:/usr/bin# ln -s g++-2.95 g++
chswj:/usr/bin#
chswj:/usr/bin# ls -l | grep -E "gcc|g\+\+"
-rwxr-xr-x  1 root root       428 May  7  2006 c89-gcc
-rwxr-xr-x  1 root root       451 May  7  2006 c99-gcc
lrwxrwxrwx  1 root root         8 May 27 07:33 g++ -> g++-2.95
-rwxr-xr-x  1 root root     74244 Jul 13  2006 g++-2.95
-rwxr-xr-x  1 root root    183444 Dec 10  2006 g++-4.1
lrwxrwxrwx  1 root root         8 May 27 07:33 gcc -> gcc-2.95
-rwxr-xr-x  1 root root     74104 Jul 13  2006 gcc-2.95
-rwxr-xr-x  1 root root    183444 Dec 10  2006 gcc-4.1
lrwxrwxrwx  1 root root        10 May 26 18:07 gccbug -> gccbug-4.1
-rwxr-xr-x  1 root root     16283 Dec 10  2006 gccbug-4.1
lrwxrwxrwx  1 root root         7 May 26 18:07 i486-linux-gnu-g++ -> g++-4.1
lrwxrwxrwx  1 root root         7 May 26 18:07 i486-linux-gnu-g++-4.1 -> g++-4.1
lrwxrwxrwx  1 root root         7 May 26 18:07 i486-linux-gnu-gcc -> gcc-4.1
lrwxrwxrwx  1 root root         7 May 26 18:07 i486-linux-gnu-gcc-4.1 -> gcc-4.1
chswj:/usr/bin#
{% endhighlight %}
Now I am going to create a build directory to have everything in one place
{% highlight bash %}
chswj:~#
chswj:~# export MYBUILDENV="/root/buildenv"
chswj:~# mkdir $MYBUILDENV
chswj:~#
{% endhighlight %}

