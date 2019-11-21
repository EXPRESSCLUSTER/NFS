# NFS on Windows 
- This article describes how to create NFS cluster on Windows Server.

## Index
- [Overview](#overview)
- [Evaluation Environment](#evaluation-environment)
- [Modify Registry Keys](#modify-registry-keys)
- [Create NFS Cluster](#create-nfs-cluster)

## Overview
```
      +----------------------+
      | NFS Server #1        |   +------------------+
      | Windows Server 2016  |   | Mirror Disk      |
  +---+  Server for NFS      +---+  NFS directories |
  |   |  EXPRESSCLUSTER X    |   |                  |
  |   +----------------------+   +---A--------------+
  |                                  | Mirroring 
  |   +----------------------+       |
  |   | NFS Server #2        |   +---V--------------+
  |   |  Windows Server 2016 |   | Mirror Disk      |
  +---+  Server for NFS      +---+  NFS directories |  
  |   |  EXPRESSCLUSTER X    |   |                  |
  |   +----------------------+   +------------------+
  |
  |   +----------------------+
  |   | NFS Client           |
  +---+  CentOS 7.7          |
      |                      |
      +----------------------+
```
- Mirror Disk Resource
  - Contains NFS directories and files. Replicates the files to the standby server.
- Script Resource
  - Contorls **Server for NFS** service.
- Floating IP Resource
  - Provides virtual IP address for NFS client machines to access NFS service.

## Evaluation Environment
- NFS Server
  - Windows Server 2016 Datacenter
  - EXPRESSCLUSTER X 3.3 (internal version: 11.35)
- NFS Client
  - CentOS Linux release 7.7.1908

## Modify Registry Keys


## Create NFS Cluster
1. Install **NFS Server** role on the cluster servers.
1. Install EXPRESSCLUSTER.
1. Add the following resources.
   - Floating IP Address
   - Mirror Disk Resource
   - Script Resource #1
   - Script Resource #2 
