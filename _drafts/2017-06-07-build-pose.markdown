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

