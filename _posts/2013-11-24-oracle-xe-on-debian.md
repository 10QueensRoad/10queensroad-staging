---
layout: post
title: "How to install Oracle XE 11g on Debian"
tags: [linux, oracle, database]
author: "Bernie Schelberg"
---
{% include JB/setup %}

I recently needed to install Oracle Database XE on my Debian-based GNU/Linux devlopment machine for the project I'm working on. Oracle Database XE is not supported on Debian, and there's a couple of quirks to the installation process that can be really difficult to figure out.

<!--end excerpt-->

When I started writing this post I was going to write the instructions as if you were starting from scratch. Then I realised that you probably haven't made it here because you are about to install oracle-xe, but because you've tried and have been searching for help on one error message or the other. So I'm writing approximately the steps I followed (with the exclusion of quite a few dead ends), and hopefully this helps you out of your situation.

> ## I strongly recommend that now that you've made it here, read the post through to the end and understand the process instead of just following what I've done. It will save you time in the end.

I'm starting with a minimal installation, and with my user added to the sudo group.

After downloading the installation file, the obvious thing to do is to extract it and install it. For this, install unzip and alien:

    $ sudo apt-get install unzip alien

Extract the installation file:

	$ unzip oracle-xe-11.2.0-1.0.x86_64.rpm.zip

This creates a `Disk1` directory, with the RPM package in it. This is where alien comes in. Alien converts different Linux package distribution file formats to Debian. Convert and install the package:

    $ sudo alien -i --scripts Disk1/oracle-xe-11.2.0-1.0.x86_64.rpm

You will probably see an error message like:

    /var/lib/dpkg/info/oracle-xe.postinst: line 114: /sbin/chkconfig: No such file or directory

You can ignore this, Debian still starts and stops the database in the correct sequence at startup, as far as I can tell. If you do have problems, you can [add dependency information to the script](https://wiki.debian.org/LSBInitScripts/).

The script tells you to configure oracle-xe, so let's do that:

    $ sudo /etc/init.d/oracle-xe configure

This results in a number of error messages, these we need to deal with. First of all:

    /etc/init.d/oracle-xe: line 405: /bin/awk: No such file or directory

Looks like the script needs awk as well. Debian has 3 options, I used gawk:

    $ sudo apt-get install gawk

You'll also need to add a link in `/bin` as Debian only creates a link in `/usr/bin`:

    $ sudo ln -s /usr/bin/awk /bin/awk

The next error is:

    touch: cannot touch `/var/lock/subsys/listener': No such file or directory

Thanks to Mike Smithers for [the answer to this one](http://mikesmithers.wordpress.com/2011/11/26/installing-oracle-11gxe-on-mint-and-ubuntu/). On Debian distros the directory is /var/lock, not /var/lock/subsys. Fortunately, this is easy to change by substituting the directory in the script:

    $ sudo sed -i 's,/var/lock/subsys,/var/lock,' /etc/init.d/oracle-xe

Next are the messages about missing files or directories:

    grep: /u01/app/oracle/product/11.2.0/xe/config/log/*.log: No such file or directory
    grep: /u01/app/oracle/product/11.2.0/xe/config/log/*.log: No such file or directory
    /bin/chmod: cannot access `/u01/app/oracle/diag': No such file or directory

These are a result of not having libaio installed (the error messages informing you of this are, unfortunately, sent to `/dev/null`). On Debian this package is called libaio1, so install it now:

    $ sudo apt-get install libaio1

If you try to run the configure command again, then you get the error message:

    Oracle Database 11g Express Edition is already configured

This message is printed when a file exists at `/etc/default/oracle-xe`. So delete that and run the configure again:

    $ sudo rm /etc/default/oracle-xe
    $ sudo service oracle-xe configure

Now you'll see the error message:

    Database Configuration failed. Look into /u01/app/oracle/product/11.2.0/xe/config/log for details

If you look at the log files in that directory, then you may notice some error messages such as:

    ORA-01034: ORACLE not available
    ORA-27101: shared memory realm does not exist

These error messages are caused by your shared memory not being at `/dev/shm`, where oracle-xe expects it. Debian now creates shared memory at `/run/shm`, which is where you'll see it if you run `df`:

    $ df -m
    Filesystem                     1M-blocks  Used Available Use% Mounted on 
    rootfs                             14824  2103     11968  15% / 
    udev                                  10     0        10   0% /dev 
    tmpfs                                 50     1        50   1% /run 
    /dev/mapper/dozer--debian-root     14824  2103     11968  15% / 
    tmpfs                                  5     0         5   0% /run/lock 
    tmpfs                                100     0       100   0% /run/shm 
    /dev/sda1                            228    20       197  10% /boot 

If you check `/dev/shm`, you'll find that there's a link there to `/run/shm`, but it seems that's not enough for oracle-xe. There's another problem here also; oracle-xe requires 2 gigabytes of shared memory, but you can see I only have 100 megabytes. Oracle's system requirements recommend 512MB of RAM, so that's how much I've allocated to this machine. The default amount of space allocated to `/run/shm` on Debian is 20% of RAM, hence I only have 100MB there. To solve both problems, we're going to explicitly create the tmpfs for shared memory at `/dev/shm`. This is the solution provided in the manual entry for `tmpfs`, referring to Oracle's buggy check. Add the following line to `/etc/fstab`:

    tmpfs		/dev/shm	tmpfs	nosuid,nodev,size=2G,mode=1777	0	0

Reboot to use the new configuration:

    $ sudo reboot

Now, running `df -m`, you should see a line something like:

    tmpfs                               2048     0      2048   0% /dev/shm

Now, try configuring oracle-xe again:

    $ sudo service oracle-xe configure

This time, you should see the message:

    Installation completed successfully.

Now try to connect with sqlplus. First source the oracle environment:

    $ . /u01/app/oracle/product/11.2.0/xe/bin/oracle_env.sh

This script will configure some environment variables for you so that you can access the database. You should probably add this to your .bashrc (I'll assume you've done this, if not you'll need to source it every time you log out or reboot).

Try to connect now:

    $ sqlplus system

After typing in your password, you will now probably see:

    ORA-01033: ORACLE initialization or shutdown in progress

This bit I was able to figure out with thanks to [this page](http://www.dbmotive.com/ora-01033-oracle-initialization-or-shutdown-in-progress/). To figure out what's going on, we're going to try to start the database again. First, you'll need to add yourself to the `dba` group:

    $ sudo usermod -a -G dba `whoami`

You'll need to log in again to be included in the new group.

Now, connect as sysdba:

    $ sqlplus / as sysdba
    SQL> alter database mount;

Gives the error message:

    ORA-00205: error in identifying control file, check alert log for more info

Hmm, let's see what the control file is:

    SQL> show parameters control_files;

This tells us the control file is `/u01/app/oracle/oradata/XE/control.dbf`. A quick check of the file system shows us that that file doesn't exist. A missing control file is well nigh impossible to recover from as far as I can gather, the solution that I discovered was to reinstall now that everything is in place:

    $ sudo service oracle-xe stop
    $ sudo rm /etc/default/configure
    $ sudo alien -i --scripts Disk1/oracle-xe-11.2.0-1.0.x86_64.rpm 
    $ sudo service oracle-xe configure

Now you should be able to connect with sqlplus, or your database tool of choice.
