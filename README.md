
# Arch Linux UTM Installation

A simple Arch Linux Integration using UTM. 

## Download Arch Linux ISO

Download the ISO - [**Arch Linux ISO**](https://wiki.archlinux.org/title/Installation_guide#Pre-installation)

## Download UTM

[**UTM homepage**](https://mac.getutm.app/)

```shell
brew install --cask utm
```

## Create UTM Linux VM

1. Create new VM
2. **Start** 
	- Select `Virtualize`
	- Click `Next` button
3. **Operating System**
	- Select `Linux`
4. **Linux**
	- **Boot ISO Image:** You recently download Arch Linux ISO
	- Click `Next` button
5. **Hardware **
	- Whatever you want.
	- Click `Next` button
6. **Storage**
	- Whatever you need. 
	- Click `Next` button
7. **Shared Directory **
	- I skip this setting
	- Click `Next` button
8. **Summary** 
	- Click `Next` button

### Edit VM Network Settings

1. Edit your **Linux VM** by right-clicking
2. Select `Network` 
3. **Network Mode**
	- Select `Emulated VLAN`
4. **Emulated Network Card**
	- Select `rlt8139`
5. Click `Save`  button


## Arch Linux Installation Guide

Start up your **Linux VM**.

### Check that the network is connected

```shell
ping archlinux.org
```

### Big Font

This is optional. I have poor eye sight, so the big font helps. 

```
setfont ter-120b
```

### Update system clock

```shell
 timedatectl set-ntp true
 
# view date
date
```

### Partition Drives

We'll only do `root` and `EFI` we're not doing anything fancy here.

```shell
# list your disks 
lsblk

# you're looking for /dev/sda
# make sure you don't nuke your data. 
# Thankfully this is just a vm. 
```

```shell
# Enter partition setup on /dev/sda
cfdisk /dev/sda
```


### Label type

> Let Use `dos` for VM install. Use `GPT` for newer systems or if your drive is larger than 2T bytes 

#### Boot Partition `/dev/sda1`

1. New 
2. 128M
3. press `b` to set boot flag

### Root `/dev/sda2/`

1. fres space
2. new 
3. default
4. primary

### Finish Partition 

1. select write 
2. enter yes, and press enter
3. quit

### Format the partitions

``` shell
# mkfs.fat -F 32 /dev/efi_system_partition
mkfs.ext4 /dev/sda1

# mkfs.ext4 /dev/root_partition
mkfs.ext4 /dev/sda2
```

```shell
# mount root
mount /dev/sda2 /mnt

# mount EFI
mount --mkdir /dev/sda1 /mnt/boot/EFI

# preview
lsblk 
```

# Installation

### The essentials

```shell
pacstrap /mnt base base-devel linux linux-firmware vim
```

### Config File system table

```shell
genfstab -U /mnt >> /mnt/etc/fstab

# view file
vim /mnt/etc/fstab
```

### Chroot

changing into the root directory of new install

```shell
arch-chroot /mnt /bin/bash
```
 
### network man

```shell
pacman -S networkmanager
```

Enable internet on startup

```shell
systemctl enable NetworkManager
```
 
### Time
 
Set time zone

```shell
# find your region and city 
ls /usr/share/zoneinfo
ls /usr/share/zoneinfo/Japan

# ln -sf /usr/share/zoneinfo/Region/City /etc/localtime
 ln -sf /usr/share/zoneinfo/America/Chicago /etc/localtime
```

hardware clock - to generate /etc/adjtime:

```shell
 hwclock --systohc
```

## Set the locale


Edit `/etc/locale.gen` and uncomment `en_US.UTF-8 UTF-8` and other needed locales. 

```shell
vim  /etc/locale.gen
```

remove the `#` in front of your locale

Generate the locales by running:

```shell
locale-gen
```

create locale configuration file

```shell
vim /etc/locale.conf
# add your locales
# LANG=en_US.UTF-8
```

## Network Config

Set the hostname

### Manually
```shell
vim /etc/hostname

# add you host name
# myhostname (arakuichi)
```

### Alt

```shell
hostnamectl set-hostname myhostname
```

## Host config

```shell
vim /etc/hosts
```

```
# add the following
127.0.0.1        localhost
::1              localhost
127.0.1.1        myhostname
```

Confirm host settings

```shell
getent hosts
```

## Setup Users 

Set root password.

```shell
passwd
``` 

Add user

```shell
useradd -m [user-name]
```

Set user password

```shell
passwd [user-name]
```


Add user to groups

```shell
usermod -aG wheel,audio,video,optical,storage [user]
```

Install `sudo`

```shell
pacman -S sudo
```

Edit user `sudo` privilages

```shell
visudo
# Edit to allow wheel group user access to any command
```

## Grub

```shell
pacman -S grub efibootmgr dosfstools os-prober mtools
```

```shell
grub-install --target=x86_64-efi --efi-directory=/boot/EFI --bootloader-id=GRUB

# grub config 
grub-mkconfig -o /boot/grub/grub.cfg
```

## Shutdown 

```shell
exit

umount -R /mnt`

shutdown 
```

Next, delete the ISO Drive from the VM. Then start up the VM.
