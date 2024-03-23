# Windows Server 2022 Lab

## Table of Contents

- [Windows Server 2022 Lab](#windows-server-2022-lab)
  - [Table of Contents](#table-of-contents)
  - [Introduction](#introduction)
  - [Network Diagram](#network-diagram)
    - [Virtual Network](#virtual-network)
    - [SIG Network](#sig-network)
    - [MYN Network](#myn-network)
  - [Projects List](#projects-list)
  - [Setup Workstation VM to manage](#setup-workstation-vm-to-manage)
  - [Setup PFSENSE as router for all networks](#setup-pfsense-as-router-for-all-networks)
  - [Setup base VM of Windows Server 2022 to clone](#setup-base-vm-of-windows-server-2022-to-clone)
  - [Implement SIG-SVR1 and SIG-SVR2 to setup as DCs](#implement-sig-svr1-and-sig-svr2-to-setup-as-dcs)
  - [Implement SIG-SVR1 And SIG-SVR2 to serve DNS](#implement-sig-svr1-and-sig-svr2-to-serve-dns)
  - [Implement SIG-SVR1 to serve DHCP and SIG-SVR2 as failover DHCP](#implement-sig-svr1-to-serve-dhcp-and-sig-svr2-as-failover-dhcp)
  - [Enable DHCP Relay on PFSENSE interfaces](#enable-dhcp-relay-on-pfsense-interfaces)
  - [Create OUs, Users, Groups and Computers](#create-ous-users-groups-and-computers)
    - [OUs](#ous)
    - [Users](#users)
    - [Groups](#groups)
  - [Implement SIG-SVR3 to serve MDT and WDS for Windows Server 2022 and Windows 10 Deployments](#implement-sig-svr3-to-serve-mdt-and-wds-for-windows-server-2022-and-windows-10-deployments)
    - [Create Prestaged computer account for SIG-SVR3](#create-prestaged-computer-account-for-sig-svr3)
    - [Setup SIG-SVR3 and join SIG.local domain](#setup-sig-svr3-and-join-siglocal-domain)
    - [Install Necessary Software for MDT to SIG-SVR3](#install-necessary-software-for-mdt-to-sig-svr3)
    - [Configure MDT Deployment For Windows 10 and Windows Server 2022](#configure-mdt-deployment-for-windows-10-and-windows-server-2022)
      - [Configure MDT for Deployments](#configure-mdt-for-deployments)
        - [Set Full permissions of DeploymentShare to Sysadmin group](#set-full-permissions-of-deploymentshare-to-sysadmin-group)
        - [Add Applications](#add-applications)
          - [Add 7Zip application](#add-7zip-application)
          - [Add virtio Drivers](#add-virtio-drivers)
        - [Import Windows 10 Image to MDT Deployment](#import-windows-10-image-to-mdt-deployment)
          - [Export Windows 10 WIM image](#export-windows-10-wim-image)
          - [Import Windows 10 image to MDT Deployment](#import-windows-10-image-to-mdt-deployment-1)
        - [Import Windows 10 VIRTIO Drivers for Storage and Network](#import-windows-10-virtio-drivers-for-storage-and-network)
        - [Create Profile for Windows 10 Drivers under Advanced Configuration](#create-profile-for-windows-10-drivers-under-advanced-configuration)
        - [Create Task Sequence for Windows 10 MDT Deployment](#create-task-sequence-for-windows-10-mdt-deployment)
          - [Create Task Sequence for Windows 10 by selecting Windows 10 Image](#create-task-sequence-for-windows-10-by-selecting-windows-10-image)
          - [Inject Drivers for network and storage](#inject-drivers-for-network-and-storage)
          - [Install 7Zip](#install-7zip)
          - [Install virtio drives](#install-virtio-drives)
          - [Enable RDP](#enable-rdp)
          - [Disable Firewall](#disable-firewall)
          - [Create Local Admin User](#create-local-admin-user)
          - [Join Domain at the end of tasks](#join-domain-at-the-end-of-tasks)
        - [Export W10 Images for MDT Deployments](#export-w10-images-for-mdt-deployments)
          - [Rules for Windows 10 Image for IT OU](#rules-for-windows-10-image-for-it-ou)
          - [Bootstraip.ini](#bootstraipini)
          - [Export only x64 image, and set name of image](#export-only-x64-image-and-set-name-of-image)
          - [Enable Monitoring](#enable-monitoring)
          - [Generate Boot Image](#generate-boot-image)
          - [Rules for Windows 10 Image for HR OU](#rules-for-windows-10-image-for-hr-ou)
          - [Bootstraip.ini](#bootstraipini-1)
          - [Export W10 Image for HR OU](#export-w10-image-for-hr-ou)
          - [Import Windows Server 2022 Image to MDT Deployment](#import-windows-server-2022-image-to-mdt-deployment)
          - [Import Storage and Network driver, and Setup Profile for Windows Server 2022 Image](#import-storage-and-network-driver-and-setup-profile-for-windows-server-2022-image)
          - [Create and Setup Task Sequence](#create-and-setup-task-sequence)
          - [Rules for Windows Server 2022 Image](#rules-for-windows-server-2022-image)
          - [Bootstraip.ini](#bootstraipini-2)
      - [Setup WDS on SIG-SVR for PXE Boot](#setup-wds-on-sig-svr-for-pxe-boot)
    - [Deploy Windows 10, and Windows Server 2022](#deploy-windows-10-and-windows-server-2022)
      - [Deploy SIG-PC1 for IT](#deploy-sig-pc1-for-it)
      - [Deploy SIG-PC2 for HR](#deploy-sig-pc2-for-hr)
      - [Deploy Windows Server 2022](#deploy-windows-server-2022)
  - [Setup Group Policies](#setup-group-policies)
    - [Setup share files and folders on SIG-SVR4 to setup network drives](#setup-share-files-and-folders-on-sig-svr4-to-setup-network-drives)
    - [Setup Network drives using single GPO](#setup-network-drives-using-single-gpo)
    - [IT users can only see HelpDesk folder as E: Drive](#it-users-can-only-see-helpdesk-folder-as-e-drive)
    - [HR users can only see HR folder as G: Drive](#hr-users-can-only-see-hr-folder-as-g-drive)

## Introduction

## Network Diagram

### Virtual Network
---

![Proxmox Network](./img/Virtual-Network.jpg)

### SIG Network

| Network | Hostname | IP Address| Description |
| --- | --- | --- | --- |
| SIG.local | SIG-SVR1 | 192.168.21.2 | DC, DNS and DHCP |
| SIG.local | SIG-SVR2 | 192.168.21.3 | DC, DNS and DHCP |
| SIG.local | SIG-SVR3 | 192.168.21.4 | MDT, WDS and WSUS|
| SIG.local | SIG-SVR4 | 192.168.22.5 | RODC for Branch |
| SIG.local | SIG-PCn | 192.168.23.0/24 | SIG Client |

### MYN Network

| Network | Hostname | IP Address| Description |
| --- | --- | --- | --- |
| MYN.local | MYN-SVR1 | 192.168.24.2 | DC, DNS and DHCP |
| MYN.local | MYN-SVR2 | 192.168.24.3 | RODC |

## Projects List

- [x] Setup PFSENSE as router for all networks
- [x] Setup base VM of Windows Server 2022 to clone
- [x] Setup Workstation VM to manage
- [x] Implement SIG-SVR1 and SIG-SVR2 to setup as DCs
- [x] Implement SIG-SVR1 And SIG-SVR2 to serve DNS
- [x] Implement SIG-SVR1 to serve DHCP and SIG-SVR2 as failover DHCP
- [x] Enable DHCP Relay on PFSENSE interfaces
- [x] Create OUs, Users and Groups
- [x] Implement SIG-SVR3 to serve MDT and WDS for Windows Server 2022 and Windows 10 Deployments
- [x] Deploy SIG-SVR4, SIG-PC1 and SIG-PC2 through WDS/MDT
- [ ] Setup Group Policies
	- [x] Setup share files and folders on SIG-SVR4 to setup network drives
	- [x] HelpDesk users can only see HelpDesk folder as E: Drive
	- [x] HR users can only see HR folder as G: Drive
	- [ ] Hide C drive on only SIG HR Users either use existing GPO or new GPO
	- [ ] Setup HelpDesk group as Local Admin on SIG-PCs
	- [ ] Group Policies Central Store

## Setup Workstation VM to manage

Windows 10 workstation VM is setup for managing all Windows Server 2022.

[Remote Desktop Connection Manager](https://learn.microsoft.com/en-us/sysinternals/downloads/rdcman) will be used for remote desktop client.

This VM has two network adapters, one for connecting from my host and one for connecting to VM networks.

![IP Addresses](./img/WorkstationVMIPAddresses.jpg)

## Setup PFSENSE as router for all networks

PFSENSE Firewall is used as router for all networks.

![PFSENSE Home Page](./img/PFSENSEHomePage.jpg)

Initially, any traffic from WAN to all LANs is blocked.

![PFSENSE Firewall Rules](./img/PFSENSEFirewallRules.jpg)

Allow all traffic between all LANs especially for WDS.

![PFSENSE LAN](./img/PFSENSELAN.jpg)

![PFSENSE LAN](./img/PFSENSEOPT1.jpg)

![PFSENSE LAN](./img/PFSENSEOPT2.jpg)

![PFSENSE LAN](./img/PFSENSEOPT3.jpg)

![PFSENSE LAN](./img/PFSENSEOPT4.jpg)

## Setup base VM of Windows Server 2022 to clone

Setup base VM of Windows Server 2022 to clone in case it is required.

Create a sysprep shortcut on desktop for sysprepping after clone.

![Base VM](./img/BaseVM.jpg)

## Implement SIG-SVR1 and SIG-SVR2 to setup as DCs

Clone Base VM for DCs.

SIG.local is used as domain name for SIG network.

Setup SIG-SVR1 and SIG-SVR2 as DCs.

Primary DNS sever will be pointed to each other and Secondary DNS server will be pointed to themselves.

![SIG-SVR1](./img/SIG-SVR1.jpg)

![SIG-SVR2](./img/SIG-SVR2.jpg)

## Implement SIG-SVR1 And SIG-SVR2 to serve DNS

SIG-SVR1 and SIG-SVR2 is setup as Name Servers as DNS role is installed on both servers.

![Name Server](./img/Name-Server.jpg)

Although names are resolved to IP Addresses, IP addresses are not resolved to names yet.

![Name Resolve](./img/Name-Resolve1.jpg)

Reverse lookup zone is created and PTR records for both servers are updated.

![Reverse Lookup](./img/Reverse-Lookup-Zone.jpg)

Both names and IP addresses are resolved.

![Name Resolve](./img/Name-Resolve2.jpg)

However, DNS server is shown as unknown.

After selecting "Obtain DNS server address automatically" for IPv6 and Default Server is shown correctly.

![DNS-IPv6](./img/DNS-IPv6.jpg)

![DNS-IPv6](./img/DNS-IPv6-1.jpg)

![NSLOOKUP](./img/NSLOOKUP.jpg)

## Implement SIG-SVR1 to serve DHCP and SIG-SVR2 as failover DHCP

SIG-SVR1 and SIG-SVR2 is setup as DHCP Servers and both servers are configured as Failover DHCP servers.

SIG-SVR1

![Failover DHCP](./img/DHCP-SVR1.jpg)

SIG-SVR2

![Failover DHCP](./img/DHCP-SVR2.jpg)

Below DHCP scopes are created.

| IP Range | Description |
| --- | --- |
| 192.168.21.100-200 | For SIG servers |
| 192.168.22.100-200 | For SIG Branch |
| 192.168.23.100-200 | For SIG PCs |

## Enable DHCP Relay on PFSENSE interfaces

DHCP relay is enabled on interfaces for those networks to provide DHCP requests from different networks.

Add IP Addresses of DHCP Servers and MDT/WDS Server

![DHCP Relay](./img/DHCP-Relay.jpg)

## Create OUs, Users, Groups and Computers

### OUs

Below OUs in red square are created.

**HO-Staff** OU is for the users and computers, and **SIG-SVR** OU is for servers.

![OUs](./img/OUs.jpg)

### Users

**Jim** user is created under HR-Staff OU

![HR Users](./img/HRUsers.jpg)

**John** and **Josh** users are created under IT-Staff OU

![IT Users](./img/ITUsers.jpg)

### Groups

**HR** group is created under HR-Staff OU and it consists of **Jim** user

![HR Group](./img/HRGroup.jpg)

**Sysadmin** group is created under IT-Staff OU and it consists of **Josh** user.

**Sysadmin** group is in **Domain Admins** group and **Helpdesk** group.

![Sysadmin Member](./img/SysadminGroupMembers.jpg)

![Sysadmin Member Of](./img/SysadminGroupMemberOf.jpg)

**Helpdesk** group is created under IT-Staff OU and it consists of **John** user and **Sysadmin** group.

![Helpdesk Member](./img/HelpdeskGroupMembers.jpg)

## Implement SIG-SVR3 to serve MDT and WDS for Windows Server 2022 and Windows 10 Deployments

### Create Prestaged computer account for SIG-SVR3

To place **SIG-SVR3** under **SIG-SVR** OU, Prestaged computer account is created.

![Prestaged Computer Account](./img/PrestagedComputerAccount.jpg)

### Setup SIG-SVR3 and join SIG.local domain

**SIG-SVR3** VM is created and joined SIG.local domain

![SIG-SVR3](./img/SIG-SVR3.jpg)

**SIG-SVR3** computer account is now showing other information.

![SIG-SVR3](./img/SIG-SVR3ComputerAccount.jpg)

![SIG-SVR3](./img/SIG-SVR3ComputerAccount1.jpg)

### Install Necessary Software for MDT to SIG-SVR3

Below software are copied to SIG-SVR3

* ADK and ADKPE offline installers
* 7Zip installer
* virtio-win-0.1.240 ISO for Proxmox VM drivers
* Windows Server 2022 and Windows 10 ISOs
* MicrosoftDeploymentToolkit_x64

![MDT Software](./img/MDTSoftware.jpg)

Installed MicrosoftDeploymentToolkit_x64, ADK and ADKPE

![MDT Software](./img/MDTSoftwareInstallation.jpg)

### Configure MDT Deployment For Windows 10 and Windows Server 2022

* Deploy Windows 10 and Windows Server 2022 through MDT
* VMs will join domain, and computer accounts should be in related OUs
* Computer name will be entered manually during deployment

#### Configure MDT for Deployments

##### Set Full permissions of DeploymentShare to Sysadmin group

![MDT Full Permission](./img/MDTPermission.jpg)

##### Add Applications

* 7Zip is for testing purpose
* virtio is drivers for Promox VM

###### Add 7Zip application

Command line for 7Zip

```
MsiExec.exe /i 7z2301-x64.msi /qn
```

![7Zip](./img/7Zip.jpg)

###### Add virtio Drivers

```
virtio-win-guest-tools.exe /S /v" /qn
```

![Virtio](./img/W10VirtioDrivers.jpg)

##### Import Windows 10 Image to MDT Deployment

Windows 10 latest versions do not have WIM Image and need to export WIM Image from ESD

###### Export Windows 10 WIM image

Convert ESD to WIM from Windows 10 Image

* Check Index Number for Windows 10 Edition

  * Open command prompt or PowerShell as administrator

```
dism /Get-WimInfo /WimFile:D:\Sources\install.esd
```

* Convert to WIM Image

```
dism /export-image /SourceImageFile:D:\Sources\install.esd /SourceIndex:6 /DestinationImageFile:C:\W10.wim /Compress:max /CheckIntegrity
```

![WIM Image](./img/WIMImage.jpg)

Windows 10 WIM image is exported

![Windows 10 WIM](./img/W10WIM.jpg)

###### Import Windows 10 image to MDT Deployment

![MDTWindows10Image](./img/MDTWindows10Image.jpg)

##### Import Windows 10 VIRTIO Drivers for Storage and Network

* To deploy through network, network driver will be needed for PXE boot
* Proxmox SCSI disk for Windows VM needs SCSI driver to detect virtual hard disk

![W10 Virtio Storage and Network Drivers](./img/W10VirtioStorageAndNetworkDrivers.jpg)

![W10 Virtio Storage and Network Drivers](./img/W10VirtioStorageAndNetworkDrivers2.jpg)

##### Create Profile for Windows 10 Drivers under Advanced Configuration

![W10 Driver Profile](./img/W10DriversProfile.jpg)

##### Create Task Sequence for Windows 10 MDT Deployment

###### Create Task Sequence for Windows 10 by selecting Windows 10 Image

![Task Sequence W10](./img/TSW101.jpg)

![Task Sequence W10](./img/TSW102.jpg)

###### Inject Drivers for network and storage

Select W10Drivers at Inject Drivers stage under Preinstall

![Task Sequence W10 Inject Drivers](./img/TSW10InjectDrivers.jpg)

###### Install 7Zip

![Task Sequence W10 7Zip](./img/TSW107Zip.jpg)

###### Install virtio drives

Add Install Applications task under Install Applications for 7Zip of State Restore

![Task Sequence W10 VIRTIO](./img/TSW10VIRTIO.jpg)

###### Enable RDP

Create task of Run Command line for enabling RDP for accessing remotely after Imaging folder

```
cmd.exe /c "reg add "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Terminal Server" /v fDenyTSConnections /t REG_DWORD /d 0 /f"
```

![EnableRDP](./img/TSW10EnableRDP.jpg)

###### Disable Firewall

Firewall will be disabled in this lab.

Create task of Run Command line for disabling firewall after Enable RDP

```
cmd.exe /c netsh advfirewall set allprofiles state off
```

![Disable Firewall](./img/TSW10DisableFirewall.jpg)

###### Create Local Admin User

Create Local Admin Group for creating new user, add new user in administrators group, set expiration and disable built-in administrator after disabling firewall task under State Restore

For creating new user, use below command

```
cmd.exe /c net user "user" "P@ssw0rd123!@#" /add /fullname:"user"
```

For adding new user to administrator group, use below command

```
cmd.exe /c net localgroup "Administrators" "user" /add
```

For setting password expiration for new user, use below command

```
cmd.exe /c WMIC USERACCOUNT WHERE "Name='user'" SET PasswordExpires=FALSE
```

Add Restart Computer task after above task

For disabling Built-in Administrator account, use below command

```
cmd.exe /c net user Administrator /active:no 
```

![Task Sequence W10 Local Admin User](./img/TSW10LocalAdminUser.jpg)

###### Join Domain at the end of tasks

Move **Recover From Domain** to end of Task Sequence to join Domain at the end of deployment

![Task Sequence W10 Domain Join](./img/TSW10DomainJoin.jpg)

##### Export W10 Images for MDT Deployments

###### Rules for Windows 10 Image for IT OU

Add rules on MDT Deployment Share for Windows 10 Deployment for IT OU

```
[Settings]
Priority=Default
Properties=MyCustomProperty

[Default]
OSInstall=Y
SkipCapture=YES
SkipAdminPassword=YES
SkipProductKey=YES
SkipComputerBackup=YES
SkipBitLocker=YES
SkipBDDWelcome=YES
SkipTimeZone=YES
TimeZoneName=Singapore Standard Time
SkipTaskSequence=YES
TaskSequenceID=W10
SkipComputerBackup=YES
SkipUserData=YES
SkipLocaleSelection=YES
UserLocale=0409:00000409
InputLocale= 0409:00000409
KeyboardLocale= 0409:00000409

SkipDomainMembership=Yes
JoinDomain=SIG.local
MachineObjectOU=OU=IT-Computer,OU=IT,OU=HO-Staff,DC=SIG,DC=local
DomainAdmin=Josh
DomainAdminDomain=SIG.local
DomainAdminPassword=P@ssw0rd123!@#

FinishAction=Reboot

EventService=http://SIG-SVR3:9800

```

###### Bootstraip.ini

```
[Settings]
Priority=Default

[Default]
DeployRoot=\\SIG-SVR3\DeploymentShare$
UserDomain=SIG
UserID=Josh
UserPassword=P@ssw0rd123!@#
SkipBDDWelcome=YES
```

###### Export only x64 image, and set name of image

![Windows 10 IT OU](./img/Windows10ITOU.jpg)

###### Enable Monitoring

![Enable Monitoring](./img/EnableMonitoring.jpg)

###### Generate Boot Image

Copy Windows 10 ITOU Image to new folder

![W10ITOUImage](./img/W10ITOUImage.jpg)

###### Rules for Windows 10 Image for HR OU

```
[Settings]
Priority=Default
Properties=MyCustomProperty

[Default]
OSInstall=Y
SkipCapture=YES
SkipAdminPassword=YES
SkipProductKey=YES
SkipComputerBackup=YES
SkipBitLocker=YES
SkipBDDWelcome=YES
SkipTimeZone=YES
TimeZoneName=Singapore Standard Time
SkipTaskSequence=YES
TaskSequenceID=W10
SkipComputerBackup=YES
SkipUserData=YES
SkipLocaleSelection=YES
UserLocale=0409:00000409
InputLocale= 0409:00000409
KeyboardLocale= 0409:00000409

SkipDomainMembership=Yes
JoinDomain=SIG.local
MachineObjectOU=OU=HR-Computer,OU=HR,OU=HO-Staff,DC=SIG,DC=local
DomainAdmin=Josh
DomainAdminDomain=SIG.local
DomainAdminPassword=P@ssw0rd123!@#

FinishAction=Reboot

EventService=http://SIG-SVR3:9800

```

###### Bootstraip.ini

```
[Settings]
Priority=Default

[Default]
DeployRoot=\\SIG-SVR3\DeploymentShare$
UserDomain=SIG
UserID=Josh
UserPassword=P@ssw0rd123!@#
SkipBDDWelcome=YES
```
###### Export W10 Image for HR OU

![Windows 10 for HR OU](./img/W10HROUImage.jpg)

###### Import Windows Server 2022 Image to MDT Deployment

Mount Windows Server 2022

![Windows Server 2022](./img/MountWS2022ISO.jpg)

Import Windows Server 2022 Image

![Windows Server 2022 Image](./img/WS2022IMG.jpg)

###### Import Storage and Network driver, and Setup Profile for Windows Server 2022 Image

![WS2022 Driver Profile](./img/WS2022DriverProfile.jpg)

###### Create and Setup Task Sequence

Create task sequence for WS 2022 by selecting Standard Server Task Sequence template.

![WS 2022 Task Sequence](./img/WS2022TaskSequence.jpg)

Inject Drivers

![Windows Server 2022 Inject Drivers](./img/WS2022InjectDrivers.jpg)

Install VIRTIO Drivers

![WS 2022 Install Virtio](./img/WS2022InstallVirtio.jpg)

Turn off Firewall Profiles

```
cmd.exe /c netsh advfirewall set allprofiles state off
```

![WS 2022 Turn Off Firewall](./img/WS2022TurnOffFirewall.jpg)

Enable RDP

```
cmd.exe /c "reg add "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Terminal Server" /v fDenyTSConnections /t REG_DWORD /d 0 /f"
```

![WS 2022 Enable RDP](./img/WS2022EnableRDP.jpg)

Move Recove From Domain to last

![WS 2022 Recover From Domain](./img/WS2022RecoverFromDomain.jpg)

###### Rules for Windows Server 2022 Image

```
[Settings]
Priority=Default
Properties=MyCustomProperty

[Default]
OSInstall=Y
SkipCapture=YES
SkipAdminPassword=YES
SkipProductKey=YES
SkipComputerBackup=YES
SkipBitLocker=YES
SkipBDDWelcome=YES
SkipTimeZone=YES
TimeZoneName=Singapore Standard Time
SkipTaskSequence=YES
TaskSequenceID=WS2022
SkipUserData=YES
SkipLocaleSelection=YES
UserLocale=0409:00000409
InputLocale= 0409:00000409
KeyboardLocale= 0409:00000409

SkipDomainMembership=Yes
JoinDomain=SIG.local
MachineObjectOU=OU=SIG-SVR,DC=SIG,DC=local
DomainAdmin=Josh
DomainAdminDomain=SIG.local
DomainAdminPassword=P@ssw0rd123!@#

FinishAction=Reboot

EventService=http://SIG-SVR3:9800

```

###### Bootstraip.ini

```
[Settings]
Priority=Default

[Default]
DeployRoot=\\SIG-SVR3\DeploymentShare$
UserDomain=SIG
UserID=Josh
UserPassword=P@ssw0rd123!@#
SkipBDDWelcome=YES
```

Export WS 2022 Image

![WS 2022 Image](./img/WS2022ExportedIMG.jpg)

#### Setup WDS on SIG-SVR for PXE Boot

Install, congifure, and integrate WDS with Active Directory

![WDS](./img/WDS.jpg)


Import all images to WDS

### Deploy Windows 10, and Windows Server 2022

#### Deploy SIG-PC1 for IT

SIG-PC1 computer account should be appeared in IT-Computer OU

Create VM for SIG-PC1 with UEFI

![SIG-PC1 VM](./img/SIG-PC1VM.jpg)

Press Enter for WDS Boot Manager

![WDS Boot Manager](./img/WDSBootManager.jpg)

Select Windows10ITOU Image and Press Enter

![W10ITOUWDSImage](./img/W10ITOUWDSImage.jpg)

Set Computer Name as SIG-PC1

![WDSSetComputerName](./img/WDSSetComputerName.jpg)

Wait for Completion

![WDSInstallation](./img/WDSInstallation.jpg)

After completion, SIG-PC1 is joined to domain.

![SIG-PC1](./img/SIG-PC1.jpg)

Computer account for SIG-PC1 is in IT-Computer OU.

![SIG-PC1](./img/SIG-PC1ITOU.jpg)

#### Deploy SIG-PC2 for HR

![SIG-PC2](./img/SIG-PC2About.jpg)

![SIG-PC2 OU](./img/SIG-PC2HROU.jpg)

#### Deploy Windows Server 2022

![SIG-SVR4](./img/SIG-SVR4.jpg)

![SIG-SVR4](./img/SIG-SVR4OU.jpg)

## Setup Group Policies

### Setup share files and folders on SIG-SVR4 to setup network drives

![Share Folder](./img/SIG-SVR4ShareFolder.jpg)

### Setup Network drives using single GPO

![Map Drive GPO](./img/MapDrive-GPO.jpg)

### IT users can only see HelpDesk folder as E: Drive

On SIG-PC1

![HelpDesk Map Drive](./img/MapDrive-HelpDesk.jpg)

![HelpDesk Map Drive](./img/MapDrive-HelpDesk-SIG-PC2.jpg)

### HR users can only see HR folder as G: Drive

![HR Map Drive](./img/MapDrive-HR.jpg)

