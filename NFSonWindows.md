# NFS Cluster on Windows 
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
1. Add the following value to disable handle singning.
   - Key: HKEY_LOCAL_MACHINE\System\CurrentControlSet\Services\NfsServer\Parameters
     - Name: HandleSigningEnabled
     - Type: REG_DWORD
     - Data: 0x0
1. Add the following values to disable Stealth Mode of Windows Firewall.
   - HKEY_LOCAL_MACHINE\Software\Policies\Microsoft\WindowsFirewall\DomainProfile
     - Name: DisableStealthMode
     - Type: REG_DWORD
     - Data: 0x1
   - HKEY_LOCAL_MACHINE\Software\Policies\Microsoft\WindowsFirewall\PrivateProfile
     - Name: DisableStealthMode
     - Type: REG_DWORD
     - Data: 0x1
   - HKEY_LOCAL_MACHINE\Software\Policies\Microsoft\WindowsFirewall\PublicProfile
     - Name: DisableStealthMode
     - Type: REG_DWORD
     - Data: 0x1
   - HKEY_LOCAL_MACHINE\Software\Policies\Microsoft\WindowsFirewall\StandardProfile
     - Name: DisableStealthMode
     - Type: REG_DWORD
     - Data: 0x1
1. Restart the cluster servers. 

## Create NFS Cluster
1. Install **NFS Server** role on the cluster servers.
1. Install EXPRESSCLUSTER X. For details, pleaser refer to Install and Configuration Guide.
1. Add the following resources.
   - Script Resource #1 (e.g. script1)
     - Dependency Tab
       - **Uncheck** [Follow the defaultdependency] and click Next.
     - Details Tab
       - Start.bat: Don't need to edit it.
       - Stop.bat: Click edit and paste the following lines.
         ```bat
         nfsadmin server start
         exit 0
         ```
   - Mirror Disk Resource (e.g. md)
     - Dependency Tab
       - **Uncheck** [Follow the defaultdependency] and select **script1** from right pane.
     - Details Tab
       - Assign driver letters for mirroring. For details, please refer to Install and Configuration Guide or Reference Guide.s
   - Script Resource #2 (e.g. script2)
     - Dependency Tab
       - **Uncheck** [Follow the defaultdependency] and select **md** from right pane.
     - Details Tab
       - Start.bat: Paste the following lines and modify nfsshare parameters.
         ```bat
         rem ***************************************
         rem Check startup attributes
         rem ***************************************
         IF "%CLP_EVENT%" == "RECOVER" GOTO RECOVER
         
         nfsadmin server start
         rem Need to modify command line
         nfsshare nfs01=s:\nfs01 -o rw root unmapped=yes encoding=ansi
         
         rem ***************************************
         rem Recovery process
         rem ***************************************
         :RECOVER
         
         exit 0
         ```
       - Stop.bat: Click edit and paste the following lines and modify nfsshare parameters.
         ```bat
         rem Need to modify command line
         nfsshare nfs01 /delete
         nfsadmin server stop
         
         exit 0
         ```
   - Floating IP Address (e.g. fip)
     - Dependency Tab
       - **Uncheck** [Follow the defaultdependency] and select **script2** from right pane.
     - Details Tab
       - Assign IP address.
         - If the cluster server network is 192.168.0.0/24, you need to assign 192.168.0.x.
