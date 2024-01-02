---
title: 'Jetbrains On NetBSD'
date: 2024-01-01T19:02:59-05:00
draft: false
tags: ["desktop"]
slug: "jetbrains-on-netbsd"
---

I do love the Jetbrains products. I use Pycharm Pro and home and at work. GoLand at home. I installed pycharm-bin for years, but that was version 2019... yuck. Recently it was updated to `2023.2.1`. However, I have a subscription for both Pycharm Pro and GoLand. I like Pycharm Pro's integration with databases. Makes life a little easier.

You can download the Jetbrains products from [Jetbrains](https://jetbrains.com). You can download the latest version of the Community Edition if you want. 

Here's what I did to get things to work.

## Download the files

1. Downloaded the latest version of the Pro edition. This puts a "tarball" in your Downloads directory.
2. Unzip the contents to the directory of your choice. I put mine in my home directory.

    ```bash
    ~> tar xavf pycharm-professional-2023.3.2.tar.gz -C ~
    ```
    
    You should have something like this in whatever directory you unzipped (untarred?) your file.
    
    ```bash
    ~> tree -d -L 1  ~>
    pycharm-2023.3.2/
    ├── bin
    ├── debug-eggs
    ├── help
    ├── jbr
    ├── lib
    ├── license
    ├── modules
    └── plugins
    ```
   
## Fixing pycharm.py

1. Next you need to address the `pycharm-2023.3.2/bin/pycharm.sh` file. Running it as is gives you errors.
2. Replace `pycharm.sh` with the file you find here, [pycharmpro2023.3.sh](https://github.com/rgeorgia/InfinityDesktop/tree/main/create_fvwm_desktop/misc/jetbrains).
3. If you try to run pycharm.py now you will get a bunch of ugly Java errors. To fix this mess we need to make sure the Linux compat layer is enabled and mount the proper Linux proc and tmpfs directories. This is done like so.

## Setting up Linux

1. You can enable the Linux compat layer in two ways.

    ```bash
    ~> sudo modload linux_compat
   
   ~> modstat | grep compat_linux
    compat_linux  exec  filesys  -  0  - compat_ossaudio,sysv_ipc,compat_util,compat_50,compat_43,exec_elf64
    ```
   To make sure Linux compat is loaded after a reboot, add it to the `/etc/modules.conf` file. If it doesn't exist, create it. 

    ```bash
    ~> cat /etc/modules.conf 
    compat_linux
    ```
2. Now you need to make sure the proper mount points are set for Linux. Add the following lines to your `/etc/fstab` file.

    ```bash
    procfs 		/emul/linux/proc procfs ro,linux
    tmpfs 		/emul/linux/dev/shm tmpfs rw,-m1777
    ```
    My `/etc/fstab` looks like this

    ```bash
    > cat /etc/fstab
    # NetBSD /etc/fstab
    # See /usr/share/examples/fstab/ for more examples.
    NAME=<snip>		/	ffs	rw		 1 1
    NAME=<snip>		none	swap	sw,dp		 0 0
    tmpfs		/tmp	tmpfs	rw,-m=1777,-s=ram%25
    kernfs		/kern	kernfs	rw
    ptyfs		/dev/pts	ptyfs	rw
    procfs		/proc	procfs	rw
    procfs 		/emul/linux/proc procfs ro,linux
    tmpfs		/var/shm	tmpfs	rw,-m1777,-sram%25
    tmpfs 		/emul/linux/dev/shm tmpfs rw,-m1777
    ```
   
## Launch Pycharm 

1. Execute the following command. (If you did not install to the $HOME directory, your location will, of course, be different).

   ```bash
   <$INSTALL_DIRECTORY>/bin/pycharm.sh
   ```

You should be able to enjoy a most excellent IDE!

![Pycharm Pro](/images/pycharmpro.png)
