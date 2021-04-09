---
title: "Complete Setup For Home Media Server!!"
date: 2019-10-15T20:00:00+08:00
tags: ["Linux", "Programming"]
categories: ["Linux"]
author: "Akash Rajvanshi"

autoCollapseToc: false
---

### What is Home Media Server??

A media server is a network device which acts as a central location for users to access media files on a network. There are three main components to any media server system:

<!--more-->

- 1. Server  - It is a smart device which ‘serves’ clients with the requested media files.
- 2. Storage device  - This is used to store all the media files such as videos, photos, computer files.
- 3. Clients  - These are devices that the end user can access the media from. Examples: TV, mobile phone, your personal computer.

All of these three components must be able to talk to each other - typically they are connected by a local area network (but it may also be connected by wireless methods).

![](https://cdn-images-1.medium.com/max/480/1*CeO0EcvQEXy23YzxR3MPdQ.png)

Finally, a media server system also needs to have appropriate media server software to allow users to actually communicate with the server and access files.

---

### Which Hardware/Software is best for Home Media Server??

We can’t buy anything to set up a media server as we will need specialized components for better performance, for instance we can’t install standard HDD (WD / Seagate Standard HDD’s) in the server instead we will need an Enterprise-grade HDD (Seagate IronWolf/ WD Red Etc) for our server.
For manually creating a home server we need :

- CPU  - High Performance Intel & AMD Processors
- RAM  - HyperX Fury 4x8GB DDR3–1600
- HDD  - Seagate IronWolf / Pro 4TB-10TB
- SSD  - Samsung 840EVO
- Motherboard  - Asrock Z77 Pro4-M
- Case  - Fractal Design Define Mini
- Cooler  - Thermalright Le Grand Macho RT
- PSU  - Seasonic 400W Fanless
- OS  - FreeNas or Ubuntu Server

For manual setup, we will need hardware like this. the major drawback of this type of Operating Systems(FreeNas & Ubuntu Server) is that they don’t have any mobile applications so we have to use third parties apps which have another level issue in setting them up and these servers consumes a tremendous amount of electricity for a home use.

To get rid of this, I would prefer Ready To Use Nas devices which are the better option for home users and small business users. As per market recommendation, The best Nas service for home/small-Business users is SYNOLOGY.

Synology has a wide variety of NAS systems according to your need.

The Best Budget NAS is a Synology NAS DS218+
In the next section, I will tell you the pros and cons of it and also I share my complete setup to you all.

---

### SYNOLOGY NAS DS218+ (Budget NAS)

Synology DiskStation DS218+ features a dual-core processor with AES-NI encryption engine and transcoding engine, providing high-speed file transfers and supporting real-time 4K transcoding. DS218+ is ideal for protecting critical assets and sharing files across different platforms. Synology DS218+ is backed by Synology’s 2-year limited warranty.

Synology DiskStation DS218+ is designed for home users or small businesses pursuing a compact and reliable shared storage solution, offering the flexibility to expand the 2 GB RAM to up to 6 GB RAM to process intensive workloads. DS218+ features a dual-core 2.0 GHz processor with a burst frequency of 2.5 GHz. With AES-NI, DS218+ delivers encrypted performance of up to 113 MB/s reading and 112 MB/s writing under RAID 1 configuration1 . DS218+ comes with three USB 3.0 ports. The hot-swappable drive tray design allows easy installation and maintenance on 3.5-inch HDDs without additional tools.

Its comes with its own operating system : DSM
It covers all the mobile operating systems such as Android, IOS and also Major Linux Distros . It holds low powered Intel Celeron which consumes less power.

Hardware Used with It :

HDD : Seagate IronWolf 2TB To 12TB

Expansion RAM :
1. Synology RAM DDR3L-1866 SO-DIMM 4GB (D3NS1866L-4G), OR
2. HyperX 4GB 1866MHz DDR3L CL11 1.35V SODIMM HyperX Impact Laptop Memory HX318LS11IB/4

---

### Setup Synology NAS DS218+ From Scratch

- First of all, made all the connection and Install the hard drives and RAM.

![](https://cdn-images-1.medium.com/max/1024/1*hDC2mG98u5WXctc7ZahrRQ.png)

#### To Find Synology NAS

After Successfully connecting the server to the home router and open the browser. there are two ways to find synology on your network :

1. Find.synology.com
2. Diskstation:5000

#### Setup Synology

Fill up your preferred username and password & setup quick-connect.

![](https://cdn-images-1.medium.com/max/1024/1*YxVuTy4yuwgZuDIgLXggdQ.png)

Some Quick Settings:

1. First of all check for DSM Updates

2. Right Top corner ➢ Personal Settings
    - Enable 2FA Settings for Account.
    - Customize Login Page as per your Need.

3. Control Panel Settings

    - Shared Folder  - Create Encrypted Folder or other Folders according to your Need
    - External Access  - Advanced ➢ Write your custom Domain ( Diskstation.example.com). we have discussed the whole setup of the custom domain in the next section and also write down your custom Ports for both (HTTP/HTTPS ) For instance, in my setup, I used 5009 for HTTP and 5010 for HTTPS.
    - Network  - DSM Settings ➢ Write down the ports that we have set in above settings and check on boxes of “Automatically Redirect HTTP connections to HTTPS” & Enable HTTPS v2
    - Security- Setup Firewall ➢ Edit the basic Rules and click on the checkboxes of service that you want in your NAS device Protection ➢ Enable DOS Protection  Account ➢ Enable Auto-Block  Advanced ➢ Enable HTTPS compression and set SSL Profile level to Modern Compatibility.
    - Other Settings - Set Themes and other user privileges settings according to your necessity

4. Package Center Settings
Package Center ➢ Settings ➢ Click on checkBox of “Synology Inc and Trusted Publishers”

    Name : SynoCommunity
    Location : [https://packages.SynoCommunity.com/](https://packages.SynoCommunity.com/)

    This will add Some Extra Packages to the Package Manager.

5. Some Must Packages (For Home Users)
Package Manager ➢ All Packages

    - Audio Station
    - Cloud Station Server
    - DNS Server
    - Document Viewer
    - Download Station
    - Hyper Backup
    - Mail Plus Server
    - Mail Plus
    - Photo Station
    - Video Station
    - USB Copy
    - Web Station
    - Plex
    - WordPress

![](https://cdn-images-1.medium.com/max/1024/1*2o13qqdS4xN9wpNnKvTptQ.png)

#### How To Reset Synology NAS

- Method 1 : (In this method, you won’t lose your data in NAS)
Push the pin in the reset button for 5-Sec then release pin after Bip. again insert the pin and push down for 5-Secs and release the pin after 3 Bips .
This will Restart the NAS and will show the Red Light on Status LED then connect your synology again with find.synology.com and reinstall DSM.

- Method 2 : (In this method, the data will be wiped out Completely)
Open Control Panel ➢ Update Section ➢  Reset

#### How to Setup Your Server to Your Personal Domain Name

- Purchase a domain name.

For my server, I’ve purchased a domain from NameCheap.com and have also setup personal email trial service. It will require in adding SSL to your domain.

- DNS Settings to Connect Your Domain To Your Server

Add your records to your domain. As in my setup my sub-domain is Synologyds, you can set your sub domain as you want.

- Add SSL To Your Domain Name

You can add any free SSL Certificate to your domain name but if you want a paid and trusted SSL like the one I’ve using for my domain. you can purchase it from best ssl service like Comodo DV SSL

Go to [WoTrus Certificate Store](https://store.wotrus.com) ➢ Purchase Comodo DV SSL ➢
Now go to Tool Section : 1. Fill Domain Name.
1. Download CSR TOOL.
2. Create Files `{{ Server.Key & Server.Csr }}` using CSR Tool.
3. Copy and Paste CSR to Website `{{ Tool Section }}`.
4. To Create certificate `{{ Type admin@example.com for mail}}`.
5. Use Web Mail to login your custom domain mail account.
6. Write the authentication code that you received in mail.
7. It will take 5 minutes to generate the certificate and then download it `{{ example.cert }}`.
8. Go to Synology server ➢ Security ➢ Certificates
 Then fill the details and select your certificate from your computer.

- Settings for your Synology Server

Go to Control Panel  ➢

1. External Access  ➢ Advanced
HostName : Synologyds.example.com
HTTP : 5009 ( Add Port Number You want to use your NAS over with )
HTTP/S : 5010

2. Network
DSM PORTS :
HTTP : 5009
HTTP/S : 5010
Click on check Box “ HTTP Redirect To HTTP/S ” & “ ENABLE HTTP/S V2 ”

3. Security
Certificate : We above discussed how to add a certificate ( Synlogyds.example.com )
TLS/SSL Profile Level  : Modern Compatibility

4. Web-Station
Install a package “Web\Station” from Package Center
Virtual Host : Synologyds.example.com
Port : 80/443
Protocol : HTTP / HTTPS
Backend Server : Nginx

5. DNS Server
Install this also from Package Center
ZONE ID  : synologyds.example.com
Domain Name  : Synologyds.example.com
Type : Master

6. Setup EZ-INTERNET
This will automatically detect your router and will also auto-forward the ports to your server.

![](https://cdn-images-1.medium.com/max/628/1*gOgCVJaI7Yk8zB5Vbj4BLQ.png)

7. Router Settings (M-Important)
Log-in to your router admin panel and add a Secondary DNS Server to your router settings. This secondary DNS is the Internal IP of your server (10.0.0.150)
Because without adding the secondary DNS to your router settings you won’t be able to access your server over your domain name.

#### Secure Synology

- Enable HTTP/S
- Change Default Ports To Custom Ports
- Enable and Configure Firewall
- Get SSL Certificate for Domain
- Enable 2FA
- Keep DSM Up To Date

### How To Backup Your Data In Synology NAS

#### Phone Backup

- Synology have vast variety of apps for Both Android & IOS Devices
- Download : DS FILE & DS PHOTO APP
  - DS Photo  : This app is for large no of files like 2K to 3K Photos or videos and it will automatically backup your data from your phone. This app will show all your photos and videos just like your phone’s gallery
  -  DS File  : This app is like your file manager, even in IOS devices. you can manually upload the files to a particular folder or anywhere you want on your server.

#### Laptop & PC Backup

Windows : You can take backup of your important folders automatically to this server
- Go to Settings ➢ Update and Security ➢ Backup
- Select a drive for backup
- In this step you will face a problem in selecting a drive for which you have to Download Synology Assistant App for windows then create a folder in server you want to save your backup then map this folder as a drive and then select folders you want to take backup regularly.
- You can also setup a shared folder via the Synology  Cloud Station App. You can create a folder in any drive of pc then to put anything in it which will be automatically synced to your other computers which have this cloud station installed.
- For Large Files :
  - In Windows  : You can Map a Folder as a drive , so you can easily copy your data in high speed.

  - In Linux  : There is some problem with some Linux distros to map a drive for which you can use FTP service. But there’s also one problem If you use Secure FTP (SFTP) which will cut your copying speed and that’s why you can use Unsecure FTP service for faster data speed and you must turn it off when not using it as Unsecure ftps are not secure to use.

If you want to make a backup of a particular folder of your server then you will need to install Hyper backup. This hyper backup package will let you backup your selected folder in server regularly and also allows you to backup from google drive and other different cloud vendors.

![](https://cdn-images-1.medium.com/max/1024/1*15wPYEp3fibopxdwJjzk9A.png)

You can do a vast variety of things with this small box. you can host a custom email server, a word-press website, backup module for your small business, media server, plex server, also host a custom git server and etc, etc. you can also manage your server through the mobile browser or a DS Finder App.

![](https://cdn-images-1.medium.com/max/1024/1*ALxikuTAPgBe9GoP2r56A.jpeg)

![](https://cdn-images-1.medium.com/max/1024/1*kZhY5ohYDGXowB6MfFHyIw.jpeg)

---
