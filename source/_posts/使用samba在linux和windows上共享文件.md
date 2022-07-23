# Use Samba to share files in Linux and Windows

## 1、Introduction

Samba is a free software for linking the UNIX series of operating systems with the Microsoft Windows operating system's **SMB / CIFS** (Server Message Block / Common Internet File System) network protocol. The third edition not only visits and shares SMB folders and printers, which can also be integrated into Windows Server domains, playing a domain control station (Domain Controller) and joining Active Directory members. In short, this software is a bridge between Windows and UNIX series operating systems, so that the resources are interoperable.

## 2、Install Samba Server on Linux（Ubuntu）

```
sudo apt -y update
sudo apt -y install samba
```

## 3、 Configure Samba server Share on Ubuntu 



```
sudo mkdir -p /home/share
# Access to All without Authentication
sudo chmod 777 /home/share
```
Samba uses configuration file in `/etc/samba/smb.conf`. If you change this configuration file, the changes do not take effect until you restart the Samba daemon
```
sudo vim /etc/samba/smb.conf
```
```
[global]

# Configure correct UTP
  unix charset = UTF-8
# Change this to the workgroup/NT-domain name your Samba server will be part of
   workgroup = WORKGROUP
   bind interfaces only = yes

# Set share configuration at the end
[Docs]
   path = /home/share
   writable = yes
   guest ok = yes
   guest only = yes
   create mode = 0777
   directory mode = 0777
```
## 4、Test and Config Samba Clinet 

To access a Samba share on the Linux system ,you need to install and configure Samba client .
```
sudo apt -y install smbclient cifs-utils
# test 
smbclient //sambaserver/share -U sambausername
```
## 5、 Use Windows link Samba shared folders

Open up File Explorer and then right-click on `This PC` (in the left pane). From the resulting context menu, select Add a network location (Figure A)

![](https://1123213.aiur.site/samba4window.png)

## Refer to 

> [install-and-configure-samba-server-share-on-ubuntu](https://computingforgeeks.com/install-and-configure-samba-server-share-on-ubuntu/)

>[How to connect to Linux Samba shares from Windows 10](https://www.techrepublic.com/article/how-to-connect-to-linux-samba-shares-from-windows-10/)

>[Mounting and mapping shares between Windows and Linux with Samba](https://www.redhat.com/sysadmin/samba-windows-linux)