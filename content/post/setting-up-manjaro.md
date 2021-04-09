---
title: "Setting Up Manjaro Linux From Scratch!!"
date: 2019-08-08T15:43:48+08:00
lastmod: 2019-08-08T15:43:48+08:00
draft: false
tags: ["Linux", "DevOps"]
categories: ["Linux"]
author: "Akash Rajvanshi"

autoCollapseToc: false
---

![Manjaro KDE Desktop](https://cdn-images-1.medium.com/max/1024/1*enMY7zrq6lHiaSDRwq1Qhg.png)

### STEP 1 : Download an Image File

Download an Image file from their [official\_website](https://manjaro.org/download/kde/) and install this image file through any booting software like Rufus.

<!--more-->

> NOTE : Select write in DD mode. (this option is particular for manjaro KDE version) and for other manjaro version choose “ISO image mode”

Recommended : Manjaro KDE (if your pc/laptop have i5).

Problem : Sometimes when you write your ISO in DD mode then your USB drive shows less space. like if USB is 8gb then its shows only 4mb size of whole pen drive.
To Solve this Problem  : Insert USB stick in Linux ( only Linux and MacOS can solve this Problem ).

Open KDE Partition Manager ( in Manjaro ) and delete all the partitions that are present in current USB stick, and assign new partitions (this will solve your problem).

Now install the system via USB stick.

___

## STEP 2 : Partitions

> For Example : — If You have 250GB SSD like me

> otherwise partition the drive according to your need.


```
Root (/) — Primary ( 50–60 GB)

Home (/home) — Primary (As per Need )(In my system (150 -160 GB)

Boot (/boot) — Primary ( 2–3 GB)

Extended (/Logical) — Remaining space( Total =250 Gb — 60Gb —150Gb — 3Gb = 37GB)

Var (/var) — 17GB(Because its stores all the temps and caches)

Swap(Linux Swap) — 20GB ( Mainly 2 X RAM)
```

___

## STEP 3 : Install Yaourt In Manjaro

> Yaourt, stands for Yet AnOther User Repository Tool

> Yaourt :-is a command line interface program which complete pacman for installing software on Archlinux.

### Installation:-

#### 1. Using Custom Repository.

```bash
sudo nano /etc/pacman.conf
```

add the following in the end of the file.

```bash
[archlinuxfr]
SigLevel = Never
Server = http://repo.archlinux.fr/$arch
```

save and close the file.

Now Update Repo database and install Yaourt using command.

```bash
sudo pacman -Sy yaourt
```

#### 2. Using AUR

```bash
sudo pacman -S --needed base-devel git wget yajl
```

after installing necessary dependencies we have to install package — query that allows to build and run yaourt.

```bash
git clone https://aur.archlinux.org/package-query.git

cd package-query/

makepkg -si

cd ..

git clone https://aur.archlinux.org/yaourt.git

cd yaourt/

makepkg -si

cd ..

sudo rm -dR yaourt/ package-query/
```

> Run these commands line by line .( i m not including explanation of every command in this Article. if you want to know more follow [this](https://www.ostechnix.com/install-yaourt-arch-linux/) )

This will install Yaourt in system .

Some Basics Usages of Yaourt -

- Update Arch Linux

```bash
yaourt -Syu
```

- Install a Package

```bash
yaourt -S <package-name>
```

- To Upgrade a packages

```bash
yaourt -U <package>
```

![](https://cdn-images-1.medium.com/max/845/1*LSD0D21FSotbi7n5Z48Vrw.png)<figcaption>Enable Yaourt In Octopi</figcaption>

![](https://cdn-images-1.medium.com/max/846/1*KPcDGRP6SfRjv6WWJca3Lw.png)<figcaption>This Skeleton Button is Yaourt</figcaption>

___

## STEP 4: Applications

> After Installing a system ,we will install some basic apps that we need.

> Enable AUR : — Open Octopi then go to tools and enable AUR.

#### 1. Browsers

1. Mozilla Firefox  —  by default Installed .
2. Google Chrome  —  Install Using AUR (package name — `{{ google-chrome }}` In AUR ).
3. Vivaldi  —  Install Using Yaourt lib (Click on Skeleton Button in Octopi Then search `{{ Vivaldi }}`

#### 2. Code\_Editors

1. Atom — By A
2. Web-storm  —  By Yaourt
3. VS Code Studio  —  By Yaourt (Search — `{{ Visual-studio-code-bin }}`

#### 3. For\_Download

1. uget  —  By AUR
2. Transmission(Torrent\_Client) —  AUR`{{ Transmission-cli }}`

#### 4. For\_Email

1. MailSpring  — By Yaourt

#### 5. Other\_Apps

1. Telegram\_desktop  —  By Aur
2. franz  —  By Yaourt `{{ Franz-bin }}`
3. MegaSync  —  By Yaourt `{{ megasync }}`
4. Stacer (CleanUp Tool) —  By Yaourt `{{ stacer }}`
5. TimeShift (Backup Software) - By AUR
6. Sweeper (System Cleaner)  — By AUR
7. Discover (Software Manager) —  By AUR

> Now we have all the basics apps we need.

___

## STEP 5 : Get all the apps of KDE-Plasma

> Follow this step only if you have high configurable Pc otherwise it will slow down your system or install its small package.

```bash
Sudo pacman -S Plasma kio-extras
```

- All Applications

```bash
Sudo pacman -S Kde-applications
```

- Selected Applications

```bash
Sudo pacman -S Kdebase
```

- Some other settings are for kde
SDDM(default display manager)

```bash
Sudo Systemctl enable sddm.service --force
reboot
```

- Update Current User

```bash
/usr/bin/cp -rf/etc/skel/.
reboot
```
___

## STEP 6 : System Configuration

- Desktop Theme  — Breeze dark
- Look and Feel  — Breeze dark
- Cursor  — Breeze
- Splash Screen  — Adapta
- Color  — Breath dark
- Font  — Noto sans 10
- Icons  — Breeze or La Captaine
- Application Style — Breeze

#### Widgets

- Event Calendar
- Memory Status
- Network Monitor
- System Load Viewer
- CPU Load Monitor
- Resource Monitor
- Hard Disk Space Usage
- Hard Disk I/O Monitor

#### Some Important Settings :-

Goto System Settings and in Startup and Shutdown Section

- _Click on_  — Desktop Session `{{ Restore Manually Saved Session }}`
- _Click on_ — Default Leave Option `{{ End Current Session }}`
- _Click on_ — Autostart`{{ Disable all applications }}`

Otherwise it will slow your pc and open any time new desktop.

___

## STEP 7 : Mount 2nd Storage Drive(To Increase Storage)

> This option is only for you when you have a 2nd storage drive in your system .

> Like We have two storage drives in our system 1 SSD + 1 HDD

> Then We install our system in SSD and use HDD as for media and other files of the system.


#### Partition , Formatting and Mounting

- Partition

```bash
Sudo fdisk-l
```

check the fs format of your hardisk is based on like 1TB MyHdd on /dev/sda but we can’t mount it right now , if we mount errors will come out.
we need to partition it first:

```bash
Sudo fdisk /dev/sda
```

Note  —  “ m “ for help(command list)
for checking partition table — “P”(Enter P)
To partition , enter “n”,then just choose primary by entering “p”
and then enter “1” for only one partition number.

- Formatting

Format the new partitioned HDD(remember fs format of your hdd)

```bash
Sudo mkfs.ext4 /dev/sda
```

- Mounting

Usually drives are mounted in /mnt

```bash
Sudo mkdir /mnt/sda
```

Then we can mount it by :-

```bash
Sudo mount /dev/sda /mnt/sda
```

Whenever we reboot it will automatically mounted.
we use nano /etc/fstab

Note  — To save this login terminal as a Root.

```bash
/dev/sda /mnt/sda ext4 default 0 0
```

> 1.Path for hard drive
> 2.destination for the mounted drive
> 3.format type
> 4–6. Kept as default 0 and 0 .

___

## STEP 8 : Create a Backup of system Via TimeShift

> when you are done installing all the apps then set all the settings. Now its time to create a backup of the system .
> This backup is useful if you accidentally delete any system file or any error occurred in the middle of something.

> Note -After creating Backup we will save its files in 2nd Storage drive or external hdd (because i don’t have space in my ssd to permanently saved this backup.but, if you have space in your ssd then don’t follow steps after the step 2.

Step 1. Open TimeShift

Step 2. Create Snapshot

Step 3. Now copy all the backup files in Extenal Hdd/2nd hdd.


Open Terminal As a Root

```bash
cp -a /timeShift/snapshots/{your snapshot name}/ /mnt/{Name of your 2nd storage drive}/backup
```

> Note  —  Backup from this command because without this system can’t give access to us to direct copy. and create compressed version of your backup then you will be able to save in external hdd.

for Restore  —  Open octopi and search timeShift and then install
and copy the whole snapshot to that folder Root/TimeShift/snapshots/
next, the option of restore will appear in Timeshift.

___

## STEP 9 : Clean Tmp Regularly

> Clean `/Var/tmp` folder regularly, if it is not cleared regularly ,system will crash(only when `var` folder have no space left).

- Manual Method

Open File Manager and go to Root/var folder, clean the folder according to the need and delete by the option of “Root Actions”.

We will check from terminal that how much space is occupied in `var folder`.

```bash
df -h
```

- Automated Method

Run Sweeper and Stacer Regularly.

___

## STEP 10 : Configure oh-my-Zsh Terminal

for Zsh we have to pre install these:
1.git
2.curl
3.wget

- By Curl

```bash
sudo sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```

- By pacman

```bash
pacman -S Zsh
```

Some settings for Zsh :

- Switching from  —  zsh to bash and vice versa

```bash
exec bash (to bash)
exec zsh (to zsh)
```

- Change default shell

```bash
chsh -s /bin/zsh
```

- Manual Update  

```bash
upgrade-oh-my-zsh
```

- Uninstall zsh

```bash
uninstall-oh-my-zsh
```
- Lists of shells(installed in your system)

```bash
cat /etc/shells
```

- Plugins (In my System)

```bash
plugins=(git
       archlinux
       brew
       command-not-found
       github
       npm
       node
       ruby
       ssh-agent
       sublime
       sudo
       terminalapp
       vscode
       web-search
       gem
       rvm
       bundler
       copyfile
       zsh-syntax-highlighting
       zsh-autosuggestions
       );
```

- Theme (In my system)

```bash
ZSH\_THEME=agnoster
```

- Path (Imp)

```dotfile
.Zshrc file in Home Directory

1. # Path to your oh-my-zsh installation.
  export ZSH="/home/{{ Pc\_UserName }}/.oh-my-zsh"

2. source $ZSH/oh-my-zsh.sh

3. # If you come from bash you might have to change your $PATH.
# export PATH=$HOME/bin:/usr/local/bin:$PATH
```

- Custom Plugins

  - Add Syntax highlighting

```bash
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git
${ZSH\_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
```

  - Auto Suggestion

```bash
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH\_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
```

> Run both the codes from your zsh\_Terminal

Now, Activate— go to .Zshrc file in home directory .
Add:

```bash
plugins=(
          zsh-syntax-highlighting
          zsh-autosuggestions
        );
```

- Fonts and color scheme settings

- Right click on empty _zsh\_terminal_ and click on Edit current profile then click on Appearence.

```dotfile
Color-Scheme & background - Maia
font - Meslo LG S DZ for Powerline
```

- Install- [Powerline Fonts](https://github.com/powerline/fonts)

___
---

- Some Other Guides For Zsh

[1. agnoster](https://github.com/agnoster) / [agnoster-zsh-theme](https://github.com/agnoster/agnoster-zsh-theme)

[2. powerline/fonts](https://github.com/powerline/fonts)

___
