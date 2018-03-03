---
layout: post
title:  "Mounting a remote server with sshfs"
date:   2018-02-21
categories: [osx, linux, bash, productivty]
---

Wouldn't it be nice to be able to develop directly on a remote server with all the creature comforts that you have on your local machine? While it is often good practise to develop an ML model on a small subset of data on your laptop before moving to a remote (speed of iteration, compute cost etc.) you often want to treat the remote server as an extension of your laptop environment. 

One appraoch would be to continually edit and push changes from local to remote. This however quickly becomes a drag and adds substantial friction to your workflow. By mounting a remote server on to laptop I'm able to do away with pushing and pulling and treat the remote as an extension of my local. In addition I can use all my favourite productivity tools installed on my local (eg. [Sublime Text](https://www.sublimetext.com)) direct on the remote file system. 

## Installing SSHFS

### MAC OSX

Install SSHFS on Mac OSX by downloading FUSE and SSHFS from the [osxfuse site](https://osxfuse.github.io).

### Linux

Use your standard package manager:

On Ubuntu/Debian
	
	apt-get sshfs

On Centos/Red Hat

	yum install sshfs

## Mount the Remote File System

First we need to create an empty directory to mount the file system on. Traditionally this should be a directory in the `/mnt` dir. I like to give the mount directory the name ing convention `<user>@<server>`:

	sudo mkdir -p /mnt/<user>@<server>

If your server access is password-based to mount the remote file system simply run
	
	sudo sshfs -o allow_other,defer_permissions <user>@<server>:/ /mnt/<user>@<server>

if you have set up keypairs instead run 

	sudo sshfs -o allow_other,defer_permissions,IdentityFile=~/.ssh/id_rsa <user>@<server>:/ /mnt/<user>@<server>

## Unmounting the RFS

To unmount 
	
	umount /mnt/<user>@<server>
	
## Making it easier 

To make it even easier is usally set up a couple of alias in my `~/.bashrc`

	alias mount_<server>="sudo sshfs -o allow_other,defer_permissions,IdentityFile=~/.ssh/id_rsa <user>@<server>:/ /mnt/<user>@<server>"
	alias unmount_<server>="umount /mnt/<user>@<server>"
	
You are able to mount multiple RFS at a time (obviously you need to create unique mount directories for each).
 