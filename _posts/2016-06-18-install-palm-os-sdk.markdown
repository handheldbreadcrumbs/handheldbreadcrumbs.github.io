---
layout: post
title:  "Minimalist Palm OS SDK 4.0 setup using Debian Etch on Debian Jessie"
date:   2016-06-17 00:09:56
categories: PalmOS SDK Debian Etch systemd-nspawn docker
---

In this article I have written down steps I did to setup development environment based on PalmOS SDK 4.0. This all is done in Debian Jessie Linux distribution.

{% highlight bash %}
wolf@sloth:~$ 
wolf@sloth:~$ uname -a
Linux sloth 3.16.0-4-amd64 #1 SMP Debian 3.16.7-ckt25-2 (2016-04-08) x86_64 GNU/Linux
wolf@sloth:~$ 
wolf@sloth:~$ cat /etc/debian_version 
8.4
wolf@sloth:~$ 
{% endhighlight %}

I am not using sudo, but I did most of the steps logged in as root user though. It may be so that you will need to execute most if not all commands using sudo instead as normal user.
{% highlight bash %}
# either prepended with sudo or just as root user install debootstrap
# also tool called alien is needed
apt-get install debootstrap alien
{% endhighlight %}

Before attempting to install anything set absolute path where operating system's files will be stored
{% highlight bash %}
root@sloth:/home/wolf# 
root@sloth:/home/wolf# # set absolute path where operating system's files will be stored
root@sloth:/home/wolf# export MY_CHROOT=/home/wolf/debian.4.0-etch
{% endhighlight %}

create a folder where to store all the pseudo-snapshot files
{% highlight bash %}
root@sloth:/home/wolf# mkdir $MY_CHROOT 
root@sloth:/home/wolf# 
{% endhighlight %}

now I created distribution from scrach using debootstrap
{% highlight bash %}
root@sloth:/home/wolf# 
root@sloth:/home/wolf# debootstrap --arch i386 etch $MY_CHROOT http://archive.debian.org/debian/
{% endhighlight %}

you might experience such error
{% highlight bash %}
I: Retrieving Release 
I: Retrieving Release.gpg 
I: Checking Release signature
E: Release signed by unknown key (key id B5D0C804ADB11277)
root@sloth:/home/wolf# 
{% endhighlight %}

to remedy this, one needs to retrieve public keys from a key server
{% highlight bash %}
root@sloth:/home/wolf# 
root@sloth:/home/wolf# gpg --keyserver pgp.mit.edu --recv-keys B5D0C804ADB11277
gpg: requesting key ADB11277 from hkp server pgp.mit.edu
gpg: key ADB11277: public key "Etch Stable Release Key <debian-release@lists.debian.org>" imported
gpg: no ultimately trusted keys found
gpg: Total number processed: 1
gpg:               imported: 1
root@sloth:/home/wolf# gpg --fingerprint ADB11277
pub   1024D/ADB11277 2006-09-17
      Key fingerprint = 7EA3 91D7 2477 203B 58C0  4FBC B5D0 C804 ADB1 1277
uid                  Etch Stable Release Key <debian-release@lists.debian.org>

root@sloth:/home/wolf# 
{% endhighlight %}

rerun deboostrap to create pseudo-distro file structure for debian etch with i386 architecture,
it might be that correct gpg file is located here as well 
/usr/share/keyrings/debian-archive-removed-keys.gpg
{% highlight bash %}
root@sloth:/home/wolf# debootstrap --keyring=/root/.gnupg/pubring.gpg --verbose --arch i386 etch $MY_CHROOT http://archive.debian.org/debian/
I: Retrieving Release 
I: Retrieving Release.gpg 
I: Checking Release signature
I: Valid Release signature (key id 7EA391D72477203B58C04FBCB5D0C804ADB11277)
I: Retrieving Packages 
I: Validating Packages 
I: Resolving dependencies of required packages...
I: Resolving dependencies of base packages...
I: Checking component main on http://archive.debian.org/debian...
I: Retrieving libacl1 2.2.41-1
I: Validating libacl1 2.2.41-1
I: Retrieving adduser 3.102
I: Validating adduser 3.102
...
I: Configuring sysvinit...
I: Configuring debconf-i18n...
I: Configuring debconf...
I: Unpacking the base system...
...
I: Configuring apt-utils...
I: Configuring klogd...
I: Configuring tasksel-data...
I: Configuring sysklogd...
I: Configuring tasksel...
I: Base system installed successfully.
root@sloth:/home/wolf# # so far so good :)
root@sloth:/home/wolf# 
{% endhighlight %}

ok, great. let's run our pseudo-snapshot using systemd-nspawn :)

{% highlight bash %}
root@sloth:/home/wolf#
root@sloth:/home/wolf# systemd-nspawn -D debian.4.0-etch/ /bin/bash
Spawning container debian.4.0-etch on /home/wolf/debian.4.0-etch.
Press ^] three times within 1s to kill container.
/etc/localtime is not a symlink, not updating container timezone.
debian:/#
{% endhighlight %}

I am going to install minimum set of tools needed to build palm os application,
let's add debian archive repository address to apt sources.list file
{% highlight bash %}
debian:/#
debian:/# echo "deb http://archive.debian.org/debian/ etch main non-free contrib" > /etc/apt/sources.list
debian:/#
{% endhighlight %}

let's download the package lists from the archive repository and update them to get "latest" information on packages and their dependencies and install needed packages.
{% highlight bash %}
debian:/# 
debian:/# apt-get update && apt-get install -y gcc-2.95 g++-2.95 alien lynx unzip tar cogito git-core curl build-essential
{% endhighlight %}

{% highlight bash %}
Get:1 http://archive.debian.org etch Release.gpg [1033B]
Hit http://archive.debian.org etch Release
Ign http://archive.debian.org etch/main Packages/DiffIndex
Get:2 http://archive.debian.org etch/non-free Packages [101kB]
Get:3 http://archive.debian.org etch/contrib Packages [70.0kB]
Hit http://archive.debian.org etch/main Packages
Fetched 171kB in 1s (116kB/s)
Reading package lists... Done
Reading package lists... Done
Building dependency tree... Done
tar is already the newest version.
The following extra packages will be installed:
  binutils ca-certificates cpp cpp-2.95 cpp-4.1 debhelper dpkg-dev file g++ g++-4.1 gcc gcc-4.1 gettext gettext-base html2text intltool-debian libbeecrypt6 libc6-dev libcurl3
  libcurl3-gnutls libdigest-sha1-perl liberror-perl libexpat1 libidn11 libkrb53 libmagic1 libneon25 librpm4 libsqlite3-0 libssp0 libstdc++2.10-dev libstdc++2.10-glibc2.2
  libstdc++6-4.1-dev libxml2 linux-kernel-headers make openssl patch perl perl-modules po-debconf rcs rpm
Suggested packages:
  bzip2 lsb-rpm lintian binutils-doc cpp-doc gcc-4.1-locales dh-make debian-keyring gcc-2.95-doc gcc-4.1-doc lib64stdc++6 manpages-dev autoconf automake1.9 libtool flex bison gdb
  gcc-doc libc6-dev-amd64 lib64gcc1 lib64ssp0 cvs gettext-doc git-arch git-cvs git-svn git-email git-daemon-run gitk gitweb glibc-doc krb5-doc krb5-user stl-manual libstdc++6-4.1-doc
  make-doc-non-dfsg diff-doc libterm-readline-gnu-perl libterm-readline-perl-perl zip
Recommended packages:
  gawk libc-dev libmudflap0-dev git-doc less rsync ssh-client python xml-core mime-support perl-doc libmail-sendmail-perl libcompress-zlib-perl
The following NEW packages will be installed:
  alien binutils build-essential ca-certificates cogito cpp cpp-2.95 cpp-4.1 curl debhelper dpkg-dev file g++ g++-2.95 g++-4.1 gcc gcc-2.95 gcc-4.1 gettext gettext-base git-core
  html2text intltool-debian libbeecrypt6 libc6-dev libcurl3 libcurl3-gnutls libdigest-sha1-perl liberror-perl libexpat1 libidn11 libkrb53 libmagic1 libneon25 librpm4 libsqlite3-0
  libssp0 libstdc++2.10-dev libstdc++2.10-glibc2.2 libstdc++6-4.1-dev libxml2 linux-kernel-headers lynx make openssl patch perl perl-modules po-debconf rcs rpm unzip
0 upgraded, 52 newly installed, 0 to remove and 0 not upgraded.
Need to get 37.1MB of archives.
After unpacking 126MB of additional disk space will be used.
Get:1 http://archive.debian.org etch/main libmagic1 4.17-5etch3 [275kB]
Get:2 http://archive.debian.org etch/main file 4.17-5etch3 [31.9kB]
...
Get:31 http://archive.debian.org etch/main libstdc++6-4.1-dev 4.1.1-21 [1634kB]                                                                                                          
Get:32 http://archive.debian.org etch/main g++-4.1 4.1.1-21 [2615kB]                                                                                                                     
...
Get:52 http://archive.debian.org etch/main unzip 5.52-9etch1 [152kB]                                                                                                                     
Fetched 37.1MB in 18s (2032kB/s)                                                                                                                                                         
dpkg-preconfigure: unable to re-open stdin: 
Selecting previously deselected package libmagic1.
(Reading database ... 7194 files and directories currently installed.)
Unpacking libmagic1 (from .../libmagic1_4.17-5etch3_i386.deb) ...
Selecting previously deselected package file.
Unpacking file (from .../file_4.17-5etch3_i386.deb) ...
Selecting previously deselected package gettext-base.
Unpacking gettext-base (from .../gettext-base_0.16.1-1_i386.deb) ...
...
Setting up libmagic1 (4.17-5etch3) ...
Setting up gcc-4.1 (4.1.1-21) ...
Setting up gcc (4.1.1-15) ...
Setting up openssl (0.9.8c-4etch9) ...
Setting up ca-certificates (20070303) ...
Updating certificates in /etc/ssl/certs....done.
...
Setting up cogito (0.18.2-1) ...
debian:/# 
{% endhighlight %}

now I am going to "fix" symbolic link to gcc-4.1, for palm resource compiler and palm os app building we need gcc-2.95, current link should be removed and relinked to gcc-2.95.

{% highlight bash %}
debian:/# 
debian:/# cd /usr/bin && ls -la *gcc* && echo "--------------------" && rm gcc && ln -s gcc-2.95 gcc && ls -la *gcc*
-rwxr-xr-x 1 root root    428 May  7  2006 c89-gcc
-rwxr-xr-x 1 root root    451 May  7  2006 c99-gcc
lrwxrwxrwx 1 root root      7 Jun 15 21:33 gcc -> gcc-4.1
-rwxr-xr-x 1 root root  74104 Jul 13  2006 gcc-2.95
-rwxr-xr-x 1 root root 183444 Dec 10  2006 gcc-4.1
lrwxrwxrwx 1 root root     10 Jun 15 21:33 gccbug -> gccbug-4.1
-rwxr-xr-x 1 root root  16283 Dec 10  2006 gccbug-4.1
lrwxrwxrwx 1 root root      7 Jun 15 21:33 i486-linux-gnu-gcc -> gcc-4.1
lrwxrwxrwx 1 root root      7 Jun 15 21:33 i486-linux-gnu-gcc-4.1 -> gcc-4.1
--------------------
-rwxr-xr-x 1 root root    428 May  7  2006 c89-gcc
-rwxr-xr-x 1 root root    451 May  7  2006 c99-gcc
lrwxrwxrwx 1 root root      8 Jun 15 21:38 gcc -> gcc-2.95
-rwxr-xr-x 1 root root  74104 Jul 13  2006 gcc-2.95
-rwxr-xr-x 1 root root 183444 Dec 10  2006 gcc-4.1
lrwxrwxrwx 1 root root     10 Jun 15 21:33 gccbug -> gccbug-4.1
-rwxr-xr-x 1 root root  16283 Dec 10  2006 gccbug-4.1
lrwxrwxrwx 1 root root      7 Jun 15 21:33 i486-linux-gnu-gcc -> gcc-4.1
lrwxrwxrwx 1 root root      7 Jun 15 21:33 i486-linux-gnu-gcc-4.1 -> gcc-4.1
debian:/usr/bin# 
{% endhighlight %}


now I am going to create a folder where to store additional tools that will be installed for palm os environment.

{% highlight bash %}
debian:/usr/bin# 
debian:/usr/bin# mkdir /root/install
debian:/usr/bin#    
{% endhighlight %}    

let's move to this new folder
{% highlight bash %}
debian:/usr/bin# cd /root/install/
debian:~/install# 
{% endhighlight %}

additional tools are stored in git repository, let's clone it
{% highlight bash %}
debian:~/install# git clone git://github.com/wolf3d/palmdev
remote: Counting objects: 45, done.
Indexing 45 objects.
remote: Total 45 (delta 0), reused 0 (delta 0), pack-reused 45
 100% (45/45) done
Resolving 11 deltas.
 100% (11/11) done
debian:~/install# 
{% endhighlight %}

let's move to folder where palm sdk is located and install it with alien, since sdk is rpm package
{% highlight bash %}
debian:~/install# cd palmdev/setup.4.0/
debian:~/install/palmdev/setup.4.0# # untar it first :)
debian:~/install/palmdev/setup.4.0# 
debian:~/install/palmdev/setup.4.0# tar xvzf sdk40.tar.gz
palmos-sdk-4.0-1.noarch.rpm
Palm OS SDK Licenses/
Palm OS SDK Licenses/Palm License.txt
Documentation/Palm OS 4.0 SDK ReadMe.txt
debian:~/install/palmdev/setup.4.0# 
debian:~/install/palmdev/setup.4.0# alien -i -v palmos-sdk-4.0-1.noarch.rpm
    LANG=C rpm -qp --queryformat %{SUMMARY} palmos-sdk-4.0-1.noarch.rpm
    LANG=C rpm -qp --queryformat %{POSTIN} palmos-sdk-4.0-1.noarch.rpm
    LANG=C rpm -qp --queryformat %{NAME} palmos-sdk-4.0-1.noarch.rpm
    LANG=C rpm -qp --queryformat %{POSTUN} palmos-sdk-4.0-1.noarch.rpm
    LANG=C rpm -qp --queryformat %{PREUN} palmos-sdk-4.0-1.noarch.rpm
    ...
    chmod 644 palmos-sdk-4.0//opt/palmdev/sdk-4/include/BuildDefines.h
    chown 0:0 palmos-sdk-4.0//opt/palmdev/sdk-4/include/Core
    chmod 755 palmos-sdk-4.0//opt/palmdev/sdk-4/include/Core
    chmod 644 palmos-sdk-4.0//opt/palmdev/sdk-4/include/Core/UI/GraffitiReference.h
    chown 0:0 palmos-sdk-4.0//opt/palmdev/sdk-4/include/Core/UI/GraffitiShift.h
    chmod 644 palmos-sdk-4.0//opt/palmdev/sdk-4/include/Core/UI/GraffitiShift.h
    chown 0:0 palmos-sdk-4.0//opt/palmdev/sdk-4/include/Core/UI/InsPoint.h
    ...
    chmod 644 palmos-sdk-4.0//opt/palmdev/sdk-4/include/Core/UI/InsPoint.h
    chown 0:0 palmos-sdk-4.0//opt/palmdev/sdk-4/include/Core/UI/Keyboard.h
    chown 0:0 palmos-sdk-4.0//opt/palmdev/sdk-4/include/Libraries/PalmOSGlue/OmGlue.h
    chmod 644 palmos-sdk-4.0//opt/palmdev/sdk-4/include/Libraries/PalmOSGlue/OmGlue.h
    chown 0:0 palmos-sdk-4.0//opt/palmdev/sdk-4/include/Libraries/PalmOSGlue/PalmOSGlue.h
    ...
    chown 0:0 palmos-sdk-4.0//opt/palmdev/sdk-4/lib
    chmod 755 palmos-sdk-4.0//opt/palmdev/sdk-4/lib
    chown 0:0 palmos-sdk-4.0//opt/palmdev/sdk-4/lib/m68k-palmos-coff
    chmod 755 palmos-sdk-4.0//opt/palmdev/sdk-4/lib/m68k-palmos-coff
    chown 0:0 palmos-sdk-4.0//opt/palmdev/sdk-4/lib/m68k-palmos-coff/libNetSocket.a
    chmod 644 palmos-sdk-4.0//opt/palmdev/sdk-4/lib/m68k-palmos-coff/libNetSocket.a
    chown 0:0 palmos-sdk-4.0//opt/palmdev/sdk-4/lib/m68k-palmos-coff/libPalmOSGlue-debug.a
    chmod 644 palmos-sdk-4.0//opt/palmdev/sdk-4/lib/m68k-palmos-coff/libPalmOSGlue-debug.a
    chown 0:0 palmos-sdk-4.0//opt/palmdev/sdk-4/lib/m68k-palmos-coff/libPalmOSGlue.a
    chmod 644 palmos-sdk-4.0//opt/palmdev/sdk-4/lib/m68k-palmos-coff/libPalmOSGlue.a
    mkdir palmos-sdk-4.0/debian
    hostname -f
hostname: Unknown host
    822-date
    hostname -f
hostname: Host name lookup failure
    822-date
    chmod 755 palmos-sdk-4.0/debian/rules
    debian/rules binary 2>&1
    dpkg --no-force-overwrite -i palmos-sdk_4.0-2_all.deb
    find palmos-sdk-4.0 -type d -exec chmod 755 {} ;
    rm -rf palmos-sdk-4.0
debian:~/install/palmdev/setup.4.0#
{% endhighlight %}

now I am going to move sdk folder from /opt to /usr/local and create a symbolic link to sdk-4 folder named sdk
{% highlight bash %}
debian:~/install/palmdev/setup.4.0# cd /opt && mv palmdev /usr/local/
debian:/opt# cd /usr/local/palmdev && ln -s sdk-4 sdk
debian:/usr/local/palmdev# 
{% endhighlight %}

now let's go back and install prc-tools
{% highlight bash %}
debian:/usr/local/palmdev# cd /root/install/palmdev/setup.4.0
debian:~/install/palmdev/setup.4.0# alien -i -v prc-tools-2.0.90-1.Linux-i386.rpm
    LANG=C rpm -qp --queryformat %{SUMMARY} prc-tools-2.0.90-1.Linux-i386.rpm
    LANG=C rpm -qp --queryformat %{POSTIN} prc-tools-2.0.90-1.Linux-i386.rpm
    LANG=C rpm -qp --queryformat %{NAME} prc-tools-2.0.90-1.Linux-i386.rpm
    LANG=C rpm -qp --queryformat %{POSTUN} prc-tools-2.0.90-1.Linux-i386.rpm
    LANG=C rpm -qp --queryformat %{PREUN} prc-tools-2.0.90-1.Linux-i386.rpm
    LANG=C rpm -qp --queryformat %{RELEASE} prc-tools-2.0.90-1.Linux-i386.rpm
    LANG=C rpm -qp --queryformat %{PREFIXES} prc-tools-2.0.90-1.Linux-i386.rpm
    LANG=C rpm -qp --queryformat %{CHANGELOGTEXT} prc-tools-2.0.90-1.Linux-i386.rpm
    LANG=C rpm -qp --queryformat %{COPYRIGHT} prc-tools-2.0.90-1.Linux-i386.rpm
    LANG=C rpm -qp --queryformat %{DESCRIPTION} prc-tools-2.0.90-1.Linux-i386.rpm
    LANG=C rpm -qp --queryformat %{ARCH} prc-tools-2.0.90-1.Linux-i386.rpm
    LANG=C rpm -qp --queryformat %{VERSION} prc-tools-2.0.90-1.Linux-i386.rpm
    LANG=C rpm -qp --queryformat %{PREIN} prc-tools-2.0.90-1.Linux-i386.rpm
    LANG=C rpm -qcp prc-tools-2.0.90-1.Linux-i386.rpm
    rpm -qpi prc-tools-2.0.90-1.Linux-i386.rpm
    LANG=C rpm -qpl prc-tools-2.0.90-1.Linux-i386.rpm
Warning: Skipping conversion of scripts in package prc-tools: postinst prerm
Warning: Use the --scripts parameter to include the scripts.
    mkdir prc-tools-2.0.90
    chmod 755 prc-tools-2.0.90
    rpm2cpio prc-tools-2.0.90-1.Linux-i386.rpm | (cd prc-tools-2.0.90; cpio --extract --make-directories --no-absolute-filenames --preserve-modification-time) 2>&1
    chmod 755 prc-tools-2.0.90/./
    chmod 755 prc-tools-2.0.90/./usr
    chmod 755 prc-tools-2.0.90/./usr/local
    ...
    chmod 755 prc-tools-2.0.90/./usr/local/bin
    chmod 755 prc-tools-2.0.90/./usr/local/man
    chown 0:0 prc-tools-2.0.90//usr/local/bin/m68k-palmos-cpp
    chmod 755 prc-tools-2.0.90//usr/local/bin/m68k-palmos-cpp
    ...
    chown 0:0 prc-tools-2.0.90//usr/local/bin/m68k-palmos-g++
    chmod 755 prc-tools-2.0.90//usr/local/bin/m68k-palmos-g++
    chown 0:0 prc-tools-2.0.90//usr/local/bin/m68k-palmos-gasp
    chmod 755 prc-tools-2.0.90//usr/local/bin/m68k-palmos-gasp
    chown 0:0 prc-tools-2.0.90//usr/local/bin/m68k-palmos-gcc
    chmod 755 prc-tools-2.0.90//usr/local/bin/m68k-palmos-gcc
    chown 0:0 prc-tools-2.0.90//usr/local/info/gcc.info-17
    chmod 644 prc-tools-2.0.90//usr/local/info/gcc.info-17
    ...
    chown 0:0 prc-tools-2.0.90//usr/local/info/gcc.info-18
    chmod 644 prc-tools-2.0.90//usr/local/info/gcc.info-18
    chown 0:0 prc-tools-2.0.90//usr/local/info/gcc.info-19
    chmod 644 prc-tools-2.0.90//usr/local/info/gcc.info-19
    chown 0:0 prc-tools-2.0.90//usr/local/info/gcc.info-2
    chown 0:0 prc-tools-2.0.90//usr/local/info/gdb.info-3
    chmod 644 prc-tools-2.0.90//usr/local/info/gdb.info-3
    chown 0:0 prc-tools-2.0.90//usr/local/info/gdb.info-4
    chmod 644 prc-tools-2.0.90//usr/local/info/gdb.info-4
    chown 0:0 prc-tools-2.0.90//usr/local/info/gdb.info-5
    ...
    chmod 644 prc-tools-2.0.90//usr/local/info/ld.info-2
    chown 0:0 prc-tools-2.0.90//usr/local/info/ld.info-3
    chmod 644 prc-tools-2.0.90//usr/local/info/ld.info-3
    chown 0:0 prc-tools-2.0.90//usr/local/info/ld.info-4
    chmod 644 prc-tools-2.0.90//usr/local/info/ld.info-4
    chown 0:0 prc-tools-2.0.90//usr/local/info/prc-tools.info
    chmod 644 prc-tools-2.0.90//usr/local/info/prc-tools.info
    chown 0:0 prc-tools-2.0.90//usr/local/lib/gcc-lib/m68k-palmos/2.95.2-kgpd/SYSCALLS.c.X
    chmod 644 prc-tools-2.0.90//usr/local/lib/gcc-lib/m68k-palmos/2.95.2-kgpd/SYSCALLS.c.X
    chown 0:0 prc-tools-2.0.90//usr/local/lib/gcc-lib/m68k-palmos/2.95.2-kgpd/cc1
    chmod 755 prc-tools-2.0.90//usr/local/lib/gcc-lib/m68k-palmos/2.95.2-kgpd/cc1
    chown 0:0 prc-tools-2.0.90//usr/local/lib/gcc-lib/m68k-palmos/2.95.2-kgpd/cc1plus
    chmod 755 prc-tools-2.0.90//usr/local/lib/gcc-lib/m68k-palmos/2.95.2-kgpd/cc1plus
    chown 0:0 prc-tools-2.0.90//usr/local/lib/gcc-lib/m68k-palmos/2.95.2-kgpd/collect2
    chmod 755 prc-tools-2.0.90//usr/local/lib/gcc-lib/m68k-palmos/2.95.2-kgpd/collect2
    chown 0:0 prc-tools-2.0.90//usr/local/lib/gcc-lib/m68k-palmos/2.95.2-kgpd/cpp
    chmod 755 prc-tools-2.0.90//usr/local/lib/gcc-lib/m68k-palmos/2.95.2-kgpd/cpp
    chown 0:0 prc-tools-2.0.90//usr/local/m68k-palmos/bin/as
    chmod 755 prc-tools-2.0.90//usr/local/m68k-palmos/bin/as
    ...
    chmod 644 prc-tools-2.0.90//usr/local/palmdev/doc/README
    chown 0:0 prc-tools-2.0.90//usr/local/palmdev/include
    chmod 755 prc-tools-2.0.90//usr/local/palmdev/include
    chown 0:0 prc-tools-2.0.90//usr/local/palmdev/lib
    chmod 755 prc-tools-2.0.90//usr/local/palmdev/lib
    chown 0:0 prc-tools-2.0.90//usr/local/palmdev/lib/m68k-palmos-coff
    chmod 755 prc-tools-2.0.90//usr/local/palmdev/lib/m68k-palmos-coff
    mkdir prc-tools-2.0.90/debian
    hostname -f
hostname: Unknown host
    822-date
    hostname -f
hostname: Host name lookup failure
    822-date
    chmod 755 prc-tools-2.0.90/debian/rules
    debian/rules binary 2>&1
    dpkg --no-force-overwrite -i prc-tools_2.0.90-2_i386.deb
    find prc-tools-2.0.90 -type d -exec chmod 755 {} ;
    rm -rf prc-tools-2.0.90
debian:~/install/palmdev/setup.4.0#
{% endhighlight %}

let's untar palm resource compiler source and build it
{% highlight bash %}
debian:~/install/palmdev/setup.4.0# # 
debian:~/install/palmdev/setup.4.0# tar xvzf pilrc-2.8p7.tar.gz
pilrc-2.8p7/
pilrc-2.8p7/.indent.pro
pilrc-2.8p7/acinclude.m4
pilrc-2.8p7/aclocal.m4
pilrc-2.8p7/bitmap.c
pilrc-2.8p7/bitmap.h
pilrc-2.8p7/CharLatin.h
pilrc-2.8p7/chars.h
pilrc-2.8p7/configure
pilrc-2.8p7/configure.in
pilrc-2.8p7/doc/
pilrc-2.8p7/doc/2.8p6_emails/
pilrc-2.8p7/doc/2.8p6_emails/marshall.txt
...
pilrc-2.8p7/doc/2.8p6_emails/nicolas.txt
pilrc-2.8p7/example/resource.h
pilrc-2.8p7/font.c
pilrc-2.8p7/font.h
pilrc-2.8p7/fonts/
pilrc-2.8p7/fonts/pilfont.zip
pilrc-2.8p7/install-sh
pilrc-2.8p7/lex.c
pilrc-2.8p7/lex.h
pilrc-2.8p7/LICENSE.txt
pilrc-2.8p7/macres.h
...
pilrc-2.8p7/main.c
pilrc-2.8p7/pilrc.dsp
pilrc-2.8p7/pilrc.dsw
pilrc-2.8p7/pilrc.h
pilrc-2.8p7/pilrc.mak
pilrc-2.8p7/pilrc.spec
pilrc-2.8p7/plex.c
pilrc-2.8p7/plex.h
pilrc-2.8p7/README.txt
pilrc-2.8p7/ppmquant/
pilrc-2.8p7/ppmquant/palette-02.pbm
pilrc-2.8p7/ppmquant/palette-04.pgm
pilrc-2.8p7/ppmquant/palette-16.pgm
pilrc-2.8p7/ppmquant/palette-256.ppm
pilrc-2.8p7/resource.h
pilrc-2.8p7/resource.rc
pilrc-2.8p7/restype.c
pilrc-2.8p7/restype.h
pilrc-2.8p7/src2unix.sh
pilrc-2.8p7/srcindent.sh
pilrc-2.8p7/std.h
pilrc-2.8p7/util.c
pilrc-2.8p7/util.h
pilrc-2.8p7/win.c
pilrc-2.8p7/xwin.c
debian:~/install/palmdev/setup.4.0# 
{% endhighlight %}

to be able to build resource compiler I need to move to it's folder first
{% highlight bash%}
debian:~/install/palmdev/setup.4.0# 
debian:~/install/palmdev/setup.4.0# cd pilrc-2.8p7
debian:~/install/palmdev/setup.4.0/pilrc-2.8p7# ./configure && make && make install
...
creating Makefile
gcc -DPACKAGE=\"pilrc\" -DVERSION=\"2.8p7\" -DSIZEOF_SHORT=2 -DSIZEOF_INT=4 -DSIZEOF_LONG=4 -DSIZEOF_CHAR_P=4 -DUNIX=1  -I. -I.      -g -O2 -Wall -c pilrc.c
gcc -DPACKAGE=\"pilrc\" -DVERSION=\"2.8p7\" -DSIZEOF_SHORT=2 -DSIZEOF_INT=4 -DSIZEOF_LONG=4 -DSIZEOF_CHAR_P=4 -DUNIX=1  -I. -I.      -g -O2 -Wall -c lex.c
gcc -DPACKAGE=\"pilrc\" -DVERSION=\"2.8p7\" -DSIZEOF_SHORT=2 -DSIZEOF_INT=4 -DSIZEOF_LONG=4 -DSIZEOF_CHAR_P=4 -DUNIX=1  -I. -I.      -g -O2 -Wall -c util.c
gcc -DPACKAGE=\"pilrc\" -DVERSION=\"2.8p7\" -DSIZEOF_SHORT=2 -DSIZEOF_INT=4 -DSIZEOF_LONG=4 -DSIZEOF_CHAR_P=4 -DUNIX=1  -I. -I.      -g -O2 -Wall -c restype.c
gcc -DPACKAGE=\"pilrc\" -DVERSION=\"2.8p7\" -DSIZEOF_SHORT=2 -DSIZEOF_INT=4 -DSIZEOF_LONG=4 -DSIZEOF_CHAR_P=4 -DUNIX=1  -I. -I.      -g -O2 -Wall -c bitmap.c
gcc -DPACKAGE=\"pilrc\" -DVERSION=\"2.8p7\" -DSIZEOF_SHORT=2 -DSIZEOF_INT=4 -DSIZEOF_LONG=4 -DSIZEOF_CHAR_P=4 -DUNIX=1  -I. -I.      -g -O2 -Wall -c font.c
gcc -DPACKAGE=\"pilrc\" -DVERSION=\"2.8p7\" -DSIZEOF_SHORT=2 -DSIZEOF_INT=4 -DSIZEOF_LONG=4 -DSIZEOF_CHAR_P=4 -DUNIX=1  -I. -I.      -g -O2 -Wall -c plex.c
gcc -DPACKAGE=\"pilrc\" -DVERSION=\"2.8p7\" -DSIZEOF_SHORT=2 -DSIZEOF_INT=4 -DSIZEOF_LONG=4 -DSIZEOF_CHAR_P=4 -DUNIX=1  -I. -I.      -g -O2 -Wall -c makeKbd.c
gcc -DPACKAGE=\"pilrc\" -DVERSION=\"2.8p7\" -DSIZEOF_SHORT=2 -DSIZEOF_INT=4 -DSIZEOF_LONG=4 -DSIZEOF_CHAR_P=4 -DUNIX=1  -I. -I.      -g -O2 -Wall -c main.c
gcc  -g -O2 -Wall  -o pilrc  pilrc.o lex.o util.o restype.o bitmap.o font.o plex.o makeKbd.o main.o  
make[1]: Entering directory `/root/install/palmdev/setup.4.0/pilrc-2.8p7'
/bin/sh ./mkinstalldirs /usr/local/bin
  /usr/bin/install -c  pilrc /usr/local/bin/pilrc
make[1]: Nothing to be done for `install-data-am'.
make[1]: Leaving directory `/root/install/palmdev/setup.4.0/pilrc-2.8p7'
debian:~/install/palmdev/setup.4.0/pilrc-2.8p7# 
ebian:~/install/palmdev/setup.4.0/pilrc-2.8p7# 
Container debian.4.0-etch terminated by signal KILL.
root@sloth:/home/wolf# 
{% endhighlight %}

for minimal setup we are done :)
now I can import this snapshot into docker

{% highlight bash %}
root@sloth:/home/wolf# 
root@sloth:/home/wolf# tar -C debian.4.0-etch -c . | docker import - debian.4.0.etch.palmos.sdk.4.0
sha256:ffb3a9fb26751d61d90509d9726eec6339bd72ff28e20dc4c755ddc065eaa2f6
root@sloth:/home/wolf# 
root@sloth:/home/wolf# docker images
REPOSITORY                       TAG                 IMAGE ID            CREATED             SIZE
debian.4.0.etch.palmos.sdk.4.0   latest              ffb3a9fb2675        5 seconds ago       377.9 MB
root@sloth:/home/wolf# 
{% endhighlight %}

or just use this snapshot in conjunction with systemd-nspawn, 
let's compile simple example, spawn container with systemd-nspawn first
{% highlight bash %}
wolf@sloth:~$ 
wolf@sloth:~$ su # switching to root user here
Password: 
root@sloth:/home/wolf# systemd-nspawn -D ./debian.4.0-etch/ /bin/bash
Spawning container debian.4.0-etch on /home/wolf/debian.4.0-etch.
Press ^] three times within 1s to kill container.
/etc/localtime is not a symlink, not updating container timezone.
debian:/# 
{% endhighlight %}

change folder to location where hello.c file is
{% highlight bash %}
debian:/# cd /root/install/palmdev/setup.4.0/
debian:~/install/palmdev/setup.4.0# 
{% endhighlight %}

create folder for bulding
{% highlight bash %}
debian:~/install/palmdev/setup.4.0# 
debian:~/install/palmdev/setup.4.0# mkdir -p ./build
debian:~/install/palmdev/setup.4.0# 
{% endhighlight %}

copy source file to building folder
{% highlight bash %}
debian:~/install/palmdev/setup.4.0# cp ./hello.c ./build/
debian:~/install/palmdev/setup.4.0# 
debian:~/install/palmdev/setup.4.0# cd ./build/
debian:~/install/palmdev/setup.4.0/build# 
debian:~/install/palmdev/setup.4.0/build# m68k-palmos-gcc hello.c -o hello
debian:~/install/palmdev/setup.4.0/build# m68k-palmos-obj-res hello
debian:~/install/palmdev/setup.4.0/build# build-prc hello.prc "Hello, World" WRLD *.hello.grc
debian:~/install/palmdev/setup.4.0/build# 
{% endhighlight %}

let's check what do we have here
{% highlight bash %}
debian:~/install/palmdev/setup.4.0/build# ls -l
total 32
-rw-r--r-- 1 root root   24 Jun 17 23:25 code0000.hello.grc
-rw-r--r-- 1 root root  648 Jun 17 23:25 code0001.hello.grc
-rw-r--r-- 1 root root   43 Jun 17 23:25 data0000.hello.grc
-rwxr-xr-x 1 root root 2307 Jun 17 23:24 hello
-rw-r--r-- 1 root root 1058 Jun 17 23:24 hello.c
-rw-r--r-- 1 root root  859 Jun 17 23:25 hello.prc
-rw-r--r-- 1 root root   10 Jun 17 23:25 pref0000.hello.grc
-rw-r--r-- 1 root root    4 Jun 17 23:25 rloc0000.hello.grc
debian:~/install/palmdev/setup.4.0/build# 
debian:~/install/palmdev/setup.4.0/build# pwd
/root/install/palmdev/setup.4.0/build
debian:~/install/palmdev/setup.4.0/build# 
{% endhighlight %}

I opened location where build folder is in separate terminal within Jessie
{% highlight bash %}
wolf@sloth:~$ 
wolf@sloth:~$ cd debian.4.0-etch/
wolf@sloth:~/debian.4.0-etch$ cd ./root/install/palmdev/setup.4.0/build
wolf@sloth:~/debian.4.0-etch/root/install/palmdev/setup.4.0/build$ 
wolf@sloth:~/debian.4.0-etch/root/install/palmdev/setup.4.0/build$ 
wolf@sloth:~/debian.4.0-etch/root/install/palmdev/setup.4.0/build$ ls -lrta
total 40
drwxr-xr-x 6 root root 4096 Jun 18 02:24 ..
-rw-r--r-- 1 root root 1058 Jun 18 02:24 hello.c
-rwxr-xr-x 1 root root 2307 Jun 18 02:24 hello
-rw-r--r-- 1 root root    4 Jun 18 02:25 rloc0000.hello.grc
-rw-r--r-- 1 root root   10 Jun 18 02:25 pref0000.hello.grc
-rw-r--r-- 1 root root   43 Jun 18 02:25 data0000.hello.grc
-rw-r--r-- 1 root root  648 Jun 18 02:25 code0001.hello.grc
-rw-r--r-- 1 root root   24 Jun 18 02:25 code0000.hello.grc
{% endhighlight %}

here is our target prc file
{% highlight bash %}
-rw-r--r-- 1 root root  859 Jun 18 02:25 hello.prc                 
drwxr-xr-x 2 root root 4096 Jun 18 02:25 .
wolf@sloth:~/debian.4.0-etch/root/install/palmdev/setup.4.0/build$ 
{% endhighlight %}

let's run PalmOS emulator and install hello.prc
{% highlight bash %}
wolf@sloth:~/debian.4.0-etch/root/install/palmdev/setup.4.0/build$ pose
{% endhighlight %}

<div class=".gallery">
<ul id="" class="list-unstyled row">
<li class="col-xs-6 col-sm-4 col-md-3">
<a href="{{ site.url }}/assets/installpalmossdk.4.0/pose-demo-0.png" target="_blank">
<img class="{{ item.img-class }}" src="{{ site.url }}/assets/installpalmossdk.4.0/pose-demo-0.png" alt=""/>
</a>
</li>
</ul>
</div>
<div class=".gallery">
<ul id="" class="list-unstyled row">
<li class="col-xs-6 col-sm-4 col-md-3">
<a href="{{ site.url }}/assets/installpalmossdk.4.0/pose-demo-1.png" target="_blank">
<img class="{{ item.img-class }}" src="{{ site.url }}/assets/installpalmossdk.4.0/pose-demo-1.png" alt=""/>
</a>
</li>
</ul>
</div>
<div class=".gallery">
<ul id="" class="list-unstyled row">
<li class="col-xs-6 col-sm-4 col-md-3">
<a href="{{ site.url }}/assets/installpalmossdk.4.0/pose-demo-2.png" target="_blank">
<img class="{{ item.img-class }}" src="{{ site.url }}/assets/installpalmossdk.4.0/pose-demo-2.png" alt=""/>
</a>
</li>
</ul>
</div>
<div class=".gallery">
<ul id="" class="list-unstyled row">
<li class="col-xs-6 col-sm-4 col-md-3">
<a href="{{ site.url }}/assets/installpalmossdk.4.0/pose-demo-3.png" target="_blank">
<img class="{{ item.img-class }}" src="{{ site.url }}/assets/installpalmossdk.4.0/pose-demo-3.png" alt=""/>
</a>
</li>
</ul>
</div>
<div class=".gallery">
<ul id="" class="list-unstyled row">
<li class="col-xs-6 col-sm-4 col-md-3">
<a href="{{ site.url }}/assets/installpalmossdk.4.0/pose-demo-4.png" target="_blank">
<img class="{{ item.img-class }}" src="{{ site.url }}/assets/installpalmossdk.4.0/pose-demo-4.png" alt=""/>
</a>
</li>
</ul>
</div>


Reference links  
[1][http://www.tldp.org/REF/palmdevqs/index.html](http://www.tldp.org/REF/palmdevqs/index.html){:target="_blank"}
[2][http://web.archive.org/web/20080516085217/http://www.calliopeinc.com/palmprog2/tutorial/x506.html](http://web.archive.org/web/20080516085217/http://www.calliopeinc.com/palmprog2/tutorial/x506.html){:target="_blank"} - contains link to "OReilly sample project"  
[3][http://prc-tools.sourceforge.net/install/](http://prc-tools.sourceforge.net/install/){:target="_blank"}  
[4][http://linux-sxs.org/non_pc/index.html](http://linux-sxs.org/non_pc/index.html){:target="_blank"} - whole setup more or less is based on approach described here  
[5][http://technodrivel.blogspot.com/2005/09/palm-os-emulator-pose-on-debian-sarge.html](http://technodrivel.blogspot.com/2005/09/palm-os-emulator-pose-on-debian-sarge.html){:target="_blank"} - I did not do step 4 from this post though...
