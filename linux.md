# EXPRESSCLUSTER X Quick Start Guide for NFS server on Linux

Practical guide for NFSv4 cluster on CentOS 7

----

## OS settings

1. Install NFS server

		# yum -y install nfs-utils

2. Configure it to manual startup

		# systemctl disable nfs

3. Modify */etc/exports* for NFS server to export the mount point of (mirror) disk resource

## EC settings

1. Create a cluster with File Server Agent
2. Add a failover group for NFS service
3. Add a disk, md or hd resource
4. Add a execute resource

	edit start.sh / stop.sh as following

	- start.sh

			#!/bin/sh
			systemctl start nfs

	- stop.sh

			#!/bin/sh
			systemctl stop nfs

5. Add FIP resource 
6. Configure the resource dependency as following

	1. disk, md or hd resource
	2. exec resource which is defined as above.
	3. FIP resource

7. Add NFS Monitor resource