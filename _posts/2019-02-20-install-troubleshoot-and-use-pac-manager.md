---
layout: post
title: "Install, troubleshoot and use PAC Manager"
tags: PACManager SSH Linux
comments: true
---
Install and run PAC Manager in Linux machine.
<!-- more -->

<p align="center"> 
   <img alt="PAC Manager Banner" src="/assets/posts/pac-manager-banner.png" />
</p>

If you are Linux system administrator or DevOps enginerr, chances are you've got more than one server that you have to manage every day. You may even have to manage similar machines, for example there may be 4 to 8 servers in production environment. You may have to run the same command on all the machines at once. 

You can login to each maching using SSH and run commands on each one serially, or you can save yourself a lot of time and effort using some popular tools like ClusterSSH, etc.

My favourite tool is PAC Manager which is very simple and it has loads of features that really saves lot of your time and effort.

### About PAC Manager
PAC (Perl Auto Connector) Manager is a open source tool in Linux like operating system, which provides GUI (graphical user interface) to manage ssh,telnet, sftp, rdesktop, vnc,remote-tty & ftp sessions. 

### Installation
Open terminal and run the below commands in the same order

```
$ wget http://sourceforge.net/projects/pacmanager/files/pac-4.0/pac-4.5.5.5-all.deb
```

```
$ sudo dpkg -i pac-4.5.5.5-all.deb
```

![PAC Manager Dependency Error](/assets/posts/pac-manager-dependecy-error.jpg)

If you are getting any dependencies errors as below then execute the below command.

```
$ sudo apt-get install -f
```

If everything goes well and completed successfully, go to applications menu and you will find PAC available to use.

### Troubleshooting

When I installed PAC Manager in my machine it wasn't opening the application. It was just processing and killing itself after few seconds.

If you are experiencing the same error then it is probably that your machine has few packages and libraries missing to open and run SSH.

After installing the missed ones by running the below commands in terminal fixed the issue and allowed me to SSH to the servers.

```
1. apt install libglib2.0-dev libpango1.0-dev libvte-dev libvte-2.91-dev dh-make-perl
2. dh-make-perl --cpan Gnome2::Vte --build
3. sudo dpkg -i libgnome2-vte*.deb
   (if any errors, then try below to install the required packaged manually)
   sudo apt-add-repository multiverse
   sudo apt update
   sudo apt install git debhelper devscripts libvte-dev libgtk2-perl libextutils-pkgconfig-perl libextutils-depends-perl
4. sudo find /opt/pac/ -name "Vte.so*" -exec rm {} +
```
Hope this article was useful.

