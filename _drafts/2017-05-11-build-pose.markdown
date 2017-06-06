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

Before proceeding to install anything let's set variable for absolute path for chroot environment where all files will be stored
{% highlight bash %}
export MY_CHROOT=/home/$USER/debian.4.0-etch
{% endhighlight %}
create a directory where to store all environment files
{% highlight bash %}
mkdir $MY_CHROOT
{% endhighlight %}
install Debian base system
{% highlight bash %}
sudo debootstrap --variant=minbase --keyring=/usr/share/keyrings/debian-archive-removed-keys.gpg --verbose --arch i386 etch $MY_CHROOT http://archive.debian.org/debian/
{% endhighlight %}
Complete install log can be seen in this [gist](https://gist.github.com/handheldbreadcrumbs/af702fabc6f0c4407b059648faa2e5ee#file-debian-base-install-log){:target="_blank"}. After successful installation everything should be in debian.4.0-etch directory under home folder of current user, then I am running a command with a specified root directory. Specified root directory is $MY_CHROOT and command is not specified explicitly and it defaults to $SHELL, in my case it is /bin/bash, that is invoked with the -i option. 
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

{% highlight bash %}
wolf@chswj:~/Downloads$ 
wolf@chswj:~/Downloads$ sudo cp pose-3.5-2.src.rpm $MY_CHROOT/root/buildenv/
wolf@chswj:~/Downloads$ 
{% endhighlight %}

{% highlight bash %}
wolf@chswj:~$ 
wolf@chswj:~$ sudo chroot $MY_CHROOT
chswj:/# export MYBUILDENV="/root/buildenv"
chswj:/# cd $MYBUILDENV 
chswj:~/buildenv# 
chswj:~/buildenv# ls -l
total 4212
-rw-r--r-- 1 root root 4312350 May 27 16:17 pose-3.5-2.src.rpm
chswj:~/buildenv# 
{% endhighlight %}

{% highlight bash %}
chswj:~/buildenv# 
chswj:~/buildenv# mkdir pose-3.5-2.src.rpm-dir && cd pose-3.5-2.src.rpm-dir && rpm2cpio ../pose-3.5-2.src.rpm | cpio -idmv
choose-gl.diff
detect-fluid.diff
emulator_src_3.5.tar.gz
fltk-1.0.11-source.tar.gz
init-clipwidget.diff
pose.spec
separate-builddir.diff
8448 blocks
chswj:~/buildenv/pose-3.5-2.src.rpm-dir# 
{% endhighlight %}

{% highlight bash %}
chswj:~/buildenv/pose-3.5-2.src.rpm-dir# 
chswj:~/buildenv/pose-3.5-2.src.rpm-dir# tar fx emulator_src_3.5.tar.gz 
chswj:~/buildenv/pose-3.5-2.src.rpm-dir# 
chswj:~/buildenv/pose-3.5-2.src.rpm-dir# tar fx fltk-1.0.11-source.tar.gz 
chswj:~/buildenv/pose-3.5-2.src.rpm-dir# 
{% endhighlight %}

{% highlight bash %}
chswj:~/buildenv/pose-3.5-2.src.rpm-dir# 
chswj:~/buildenv/pose-3.5-2.src.rpm-dir# chmod -R u+w fltk-1.0.11 Emulator_Src_3.4
chswj:~/buildenv/pose-3.5-2.src.rpm-dir# 
{% endhighlight %}

diff files actually contained references to Emulator_Src_3.4 folder
{% highlight bash %}
chswj:~/buildenv/pose-3.5-2.src.rpm-dir#
chswj:~/buildenv/pose-3.5-2.src.rpm-dir# cd Emulator_Src_3.5
chswj:~/buildenv/pose-3.5-2.src.rpm-dir/Emulator_Src_3.5#
chswj:~/buildenv/pose-3.5-2.src.rpm-dir/Emulator_Src_3.5#
chswj:~/buildenv/pose-3.5-2.src.rpm-dir/Emulator_Src_3.5# patch -p1 < ../detect-fluid.diff
patching file BuildUnix/Makefile.am
patching file BuildUnix/configure.in
chswj:~/buildenv/pose-3.5-2.src.rpm-dir/Emulator_Src_3.5#
chswj:~/buildenv/pose-3.5-2.src.rpm-dir/Emulator_Src_3.5# patch -p1 < ../separate-builddir.diff
patching file BuildUnix/Gzip/Makefile.am
patching file BuildUnix/Makefile.am
patching file BuildUnix/espws-2.0/Makefile.am
patching file BuildUnix/jpeg/Makefile.am
chswj:~/buildenv/pose-3.5-2.src.rpm-dir/Emulator_Src_3.5# 
chswj:~/buildenv/pose-3.5-2.src.rpm-dir/Emulator_Src_3.5# patch -p1 < ../choose-gl.diff
patching file BuildUnix/configure.in
chswj:~/buildenv/pose-3.5-2.src.rpm-dir/Emulator_Src_3.5# 
chswj:~/buildenv/pose-3.5-2.src.rpm-dir/Emulator_Src_3.5# patch -p0 < ../init-clipwidget.diff
patching file SrcUnix/EmApplicationFltk.cpp
chswj:~/buildenv/pose-3.5-2.src.rpm-dir/Emulator_Src_3.5# 
{% endhighlight %}

{% highlight bash %}
chswj:~/buildenv/pose-3.5-2.src.rpm-dir/Emulator_Src_3.5/BuildUnix# 
chswj:~/buildenv/pose-3.5-2.src.rpm-dir/Emulator_Src_3.5/BuildUnix# fltk_dir=`pwd`/install-fltk
chswj:~/buildenv/pose-3.5-2.src.rpm-dir/Emulator_Src_3.5/BuildUnix# mkdir $fltk_dir
chswj:~/buildenv/pose-3.5-2.src.rpm-dir/Emulator_Src_3.5/BuildUnix# 
{% endhighlight %}
{% highlight bash %}
chswj:~/buildenv/pose-3.5-2.src.rpm-dir/Emulator_Src_3.5/BuildUnix# 
chswj:~/buildenv/pose-3.5-2.src.rpm-dir/Emulator_Src_3.5/BuildUnix# static_dir=`pwd`/static-libs
chswj:~/buildenv/pose-3.5-2.src.rpm-dir/Emulator_Src_3.5/BuildUnix# mkdir $static_dir 
chswj:~/buildenv/pose-3.5-2.src.rpm-dir/Emulator_Src_3.5/BuildUnix# 
{% endhighlight %}
{% highlight bash %}
chswj:~/buildenv/pose-3.5-2.src.rpm-dir/Emulator_Src_3.5/BuildUnix# 
chswj:~/buildenv/pose-3.5-2.src.rpm-dir/Emulator_Src_3.5/BuildUnix# ln -sf `${CXX:-g++} -print-file-name=libstdc++.a` $static_dir/libstdc++.a
chswj:~/buildenv/pose-3.5-2.src.rpm-dir/Emulator_Src_3.5/BuildUnix# 
{% endhighlight %}
{% highlight bash %}
chswj:~/buildenv/pose-3.5-2.src.rpm-dir/Emulator_Src_3.5/BuildUnix# 
chswj:~/buildenv/pose-3.5-2.src.rpm-dir/Emulator_Src_3.5/BuildUnix# mkdir -p build-normal build-profile
chswj:~/buildenv/pose-3.5-2.src.rpm-dir/Emulator_Src_3.5/BuildUnix# 
{% endhighlight %}
{% highlight bash %}
chswj:~/buildenv/pose-3.5-2.src.rpm-dir/Emulator_Src_3.5/BuildUnix# 
chswj:~/buildenv/pose-3.5-2.src.rpm-dir/Emulator_Src_3.5/BuildUnix# (cd build-normal && LDFLAGS=-L$static_dir ../BuildUnix/configure --with-fltk=$fltk_dir --disable-gl && make)
bash: ../BuildUnix/configure: No such file or directory
chswj:~/buildenv/pose-3.5-2.src.rpm-dir/Emulator_Src_3.5/BuildUnix#
{% endhighlight %}
{% highlight bash %}
chswj:~/buildenv/pose-3.5-2.src.rpm-dir/Emulator_Src_3.5/BuildUnix# 
chswj:~/buildenv/pose-3.5-2.src.rpm-dir/Emulator_Src_3.5/BuildUnix# (cd build-normal && LDFLAGS=-L$static_dir ../configure --with-fltk=$fltk_dir --disable-gl && make)
checking build system type... Invalid configuration `x86_64-unknown-linux-gnu': machine `x86_64-unknown' not recognized
configure: error: /bin/sh ../config.sub x86_64-unknown-linux-gnu failed
chswj:~/buildenv/pose-3.5-2.src.rpm-dir/Emulator_Src_3.5/BuildUnix# 
{% endhighlight %}
{% highlight bash %}
chswj:~/buildenv/pose-3.5-2.src.rpm-dir/Emulator_Src_3.5/BuildUnix# 
chswj:~/buildenv/pose-3.5-2.src.rpm-dir/Emulator_Src_3.5/BuildUnix# ls -l /usr/share/libtool/config.{guess,sub}            
lrwxrwxrwx 1 root root 20 May 26 18:07 /usr/share/libtool/config.guess -> ../misc/config.guess
lrwxrwxrwx 1 root root 18 May 26 18:07 /usr/share/libtool/config.sub -> ../misc/config.sub
chswj:~/buildenv/pose-3.5-2.src.rpm-dir/Emulator_Src_3.5/BuildUnix# 
chswj:~/buildenv/pose-3.5-2.src.rpm-dir/Emulator_Src_3.5/BuildUnix# cp /usr/share/libtool/config.{guess,sub} .
chswj:~/buildenv/pose-3.5-2.src.rpm-dir/Emulator_Src_3.5/BuildUnix# 
{% endhighlight %}

