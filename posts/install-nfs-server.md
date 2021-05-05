+++
title = "Install an Nfs Server on Ubuntu"
date = "2021-05-03T20:14:40-04:00"
author = ""
authorTwitter = "" #do not include @
cover = ""
tags = ["", ""]
keywords = ["nfs", "ubuntu", "server"]
description = "Learn to install and configure an NFS Server"
showFullContent = false
+++

NFS (Network File System) is a [distributed file system](https://en.wikipedia.org/wiki/Clustered_file_system#Distributed_file_systems) protocol which allows users on a client computer to remotely access files stored on another server or computer over a network, as though the files were locally stored on a hard drive. 

A shared filesystem eases the burden of safely sharing and storing files.  It can enable users and system administrators to consolidate resources and centralize computers or servers on a network. 

The shared directory is usually created on the machine that is to be the NFS Server and files are added to it. To gain access to this directory, the client's system mounts the shared directory on their own system.

It's worth noting that this file system was designed for Unix based systems. If you want to access files stored on a Windows machine, you should set up a Samba server, or install the NFS Client Feature on Windows Server, if that's the OS you're running. It is also worth noting that the NFS Protocol is not encrypted and does not provide user authentication. Access to the Server is restricted by IP and Hostname.

This tutorial will walk you through installing and configuring an NFS Server and Installing and Configuring the NFS Client. 

## Installing and Configuring the NFS Server

### Pre-Reqs

You'll need at least two machines running any flavor of linux or macos. One should be designated the NFS Server and the other the client. 
Make sure to jot down their IP addresses. You can find that by typing 

```ip a``` or ```ifconfig``` on your machine. 

```
NFS Server: 10.0.0.253
NFS Client: 10.0.0.0/24 range
```

## Setting up the NFS Server
The first step is to set up the server by installing necessary packages, creating and exporting the NFS directories, and configuring the firewall. 

```
sudo apt update
sudo apt install nfs-kernel-server
```

Once the installation completes, the NFS Server will start automatically. 

You can find the server configuration here:
```
/etc/default/nfs-kernel-server
```
Messing with this is out of the scope of this tutorial, so we'll leave it be. 

### Creating the NFS Directories

You can choose where this should be mounted and what you want to call it. I'm going to create it in the ```/mnt/``` directory and call it ```nfs_share```

```
mkdir -p /mnt/nfs_share
```

Now open the file ```/etc/exports``` with a text editor. I'm using nano. 

```
nano /etc/exports
```

Add the following text to the file:
```
/mnt/nfs_share 10.0.0.0/24(rw,sync,no_subtree_check,no_root_squash)
```

```
- rw enables the client machine to have read/write access to the nfs directory
- no_subtree_check speeds up transfers in the case of a partially exported volume
- no_root_squash enables users to connect with root privileges
- sync prevents data corruption in the case of a server reboot
```
Save the file and exit. 

We've just enabled and configured the sharing of the /mnt/nfs_share directory for all machines in our local subnet! 

You can add other nfs shared directories by following the same set of steps starting by making a new directory. 

### Exporting the NFS Directory
Next we'll need to export the NFS Share Directory

```
exportfs -a 
systemctl restart nfs-kernel-server
```

View the current active exports and their state:

```
exportfs -v
```

You may see extra stuff that we didn't configure here. No worries, its the default config stuff. 

### Configuring the Firewall
In order for the client to access the mounted shared directories, we'll need to configure the firewall to allow specific traffic through. 

You can do this with the help of the [Uncomplicated Firewall](https://wiki.ubuntu.com/UncomplicatedFirewall)

```
ufw allow from 10.0.0.0/24 to any port nfs
```

Reload or enable the firewall and check the status:

```
ufw enable
ufw status
```

The port ```2049``` is the port we care about. The output should show that the traffic on port ```2049``` is allowed only from the specific subnet we've specified. 

Our NFS Server is now set up! But we're not done. We'll need an NFS Client to communicate the Server. Hop on your other machine so we can finish configuration. 

## Setting up the NFS Client

Install the NFS Client on the machine you'll use to access the newly mounted shared files. 

```
sudo apt update
sudo apt install nfs-common
```

Create directories for the mount point:

```
mkdir -p /mnt/nfs_client_share
```

Mount the NFS Share in the NFS Client Share directory

```
mount 10.0.0.254:/mnt/nfs_share /mnt/nfs_client_share
```

Verify the file systems are mounted successfully by either of the following commands:

```
mount
df -h
```

Let's do another check. Navigate to the nfs_share on the server and create a random file:

```
cd /mnt/nfs_share/
touch testing1.txt
```

Now head to the NFS Client machine and check the NFS Client Share directory:

```
ls -lha /mnt/nfs_client_share/
```

You should see the file you just created. 

You're nearly there! Our client config is complete, however, It'd be helpful if we didn't have to remount this directory on reboot, right? Let's do that by editing our ```fstab``` file:

```
nano /etc/fstab
```

Add the following line:

```
10.0.0.254:/mnt/nfs_share /nfs_client_share nfs auto,noatime,nolock,bg,nfsvers=3,intr,tcp,actimeo=1800 0 0
```

This should allow the directory to remain mounted even after a reboot. If for some reason, the directory isn't mounted, all you'll have to do is run:

```
mount -a
```

To unmount at any point, for any reason, you'd just do:
```
umount /nfs_share
```

And there you have it! We now have an NFS Server configured. 