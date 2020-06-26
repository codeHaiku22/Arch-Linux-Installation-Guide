![ArchLinux Logo](https://www.archlinux.org/static/logos/archlinux-logo-dark-90dpi.ebdee92a15b3.png)

# Installation Guide
## Download ISO
Click [here](https://www.archlinux.org/download) or visit the link below to download the latest ISO image file for Arch.

[https://www.archlinux.org/download](https://www.archlinux.org/download)

## Prepare Live Bootable USB
The simplest way to prepare a live USB is via the ```dd``` command:
```bash
# dd if=/location/of/iso/file of=/device/entry/of/usb/drive
```
## Boot to USB
Insert the USB drive into the machine and boot into the USB drive.

## Connect to the Internet
_**NOTE:** Due to the lack of wifi hardware drivers and support within the Arch ISO file, a wired connection is strongly recommended._

Connect to wifi using `wifi-menu`. This will display a simple UI which will list all SSIDs and allow you to connect to a desired SSID:
```bash
# wifi-menu
```
Once connected, verify connectivity to the Internet:
```bash
# ping duckduckgo.com
```
## Update the System Clock
Use `timedatectl` command to ensure the system clock is accurate by enabling NTP:
```bash 
# timedatectl set-ntp true
```
## Disk Partitioning
_**NOTE:** The `fdisk` command has been used to partition the disks in this section._

List all existing disk and disk partitions:
```bash
# fdisk -l
```
_**NOTE:** If existing partitions exist, delete all existing paritions._

Select the desired disk to partition:
```bash
# fdisk /dev/sda
```

Verify if UEFI mode is enabled by checking for the existence of this directory:
```bash
# ls /sys/firmware/efi/efivars
```

Below are recommended partition schemes:

### UEFI
Mount Point | Partition | Parition Type | Partition Size | File System
------------|-----------|---------------|----------------|------------    
/mnt/boot or mnt/efi | /dev/sda1 | EFI System Parition | 260MB - 512MB | fat32
/mnt | /dev/sda2 | Linux x86-64 root (/) | Remainder of the device | ext4
[SWAP] | /dev/sda3 | Linux swap |  More than 512MiB | ext4

### NON-UEFI
Mount Point | Partition | Parition Type | Partition Size | File System
------------|-----------|---------------|----------------|------------   
/mnt | /dev/sda1 | Linux | Remainder of the device | ext4 
[SWAP] |  /dev/sda2 | Linux swap | More than 512 MiB | ext4

## Create File System
### UEFI
For the EFI parition type, create a fat32 file system:
```bash
# mkfs.fat -F32 /dev/sda1
```
For the root partition, create an ext4 file system:
```bash
# mkfs.ext4 /dev/sda2
```
Prepare the swap partition:
```bash
# mkswap /dev/sda3
# swapon /dev/sda3
```
### NON-UEFI
For non-UEFI parition, create an ext4 file system on the root partition:
```bash
# mkfs.ext4 /dev/sda1
```
Prepare the swap partition:
```bash
# mkswap /dev/sda2
# swapon /dev/sda2
```
## Optimize Mirrors
Sync the pacman repository:
```bash
# pacman -Syy
```
Install the `reflector` command to optimize the mirrorlist file for local mirrors:
```bash
# pacman -S reflector
```
Create a backup of the mirrorlist file:
```bash
# cp /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.bak
```
Execute the `reflector` command to optimize the mirrorlist file:
```bash
# reflector -c "US" -f 12 -l 10 -n 12 --save /etc/pacman.d/mirrorlist
```
## Install Arch Linux
Mount the root directory to the proper partition:
```bash
# mount /dev/sda1 /mnt
```
Use the `pacstrap` command to install needed packages:
```bash
# pacstrap /mnt base linux linux-firmware nano
```
## Configure the Installed Arch System
Automatically generate the fstab file using the `genfstab` command:
```bash
# genfstab -U /mnt >> /mnt/etc/fstab
```
Change root to the root directory at /mnt:
```bash
# arch-chroot /mnt
```
### Setting the Timezone

Use the `timedatectl` command to find your time zone:
```bash
# timedatectl list-timezones
```
Create a symbollic link to set the timezone (replace America/Los_Angeles with your time zone):
```bash
# ln -sf /usr/share/zoneinfo/America/Los_Angeles /etc/localtime
```
Run `hwclock` to generate /etc/adjtime:
```bash
# hwclock --systohc
```
### Setting Up Locale
_**NOTE:** Locale refers to language, number, date and currency formats  The file /etc/locale.gen contains locale settings and system languages and is commented by default._

Open the file and remove the "#" from the start of the line which contains your locale:
```bash
# nano /etc/locale.gen
```
This entry has been uncommented:
> en_US.UTF-8  

Generate the /etc/locale.conf file:
```bash
# locale-gen
# echo LANG=en_US.UTF-8  > /etc/locale.conf
# export LANG=en_US.UTF-8 
```
### Network Configuration
Create the /etc/hostname file and add the hostname entry:
```bash
# nano /etc/hostname
```
This entry has been added:
> ArchLinuxPC

Create the /etc/hosts file and add the proper entries:
```bash
# nano /etc/hosts
```
These entries have been added:
> 127.0.0.1	localhost\
> ::1		localhost\
> 127.0.1.1	ArchLinuxPC

Set up root passwd:
```bash
# passwd
```
## Install Grand Universal Bootloader (GRUB)
### UEFI
Install required packages:
```bash
# pacman -S grub efibootmgr
```
Create the directory where EFI partition will be mounted:
```bash
# mkdir /boot/efi
```
Mount the ESP partition:
```bash
# mount /dev/sda1 /boot/efi
```
Install GRUB:
```bash
# grub-install --target=x86_64-efi --bootloader-id=GRUB --efi-directory=/boot/efi
```
Generate the /boot/grub/grub.cfg file:
```bash
# grub-mkconfig -o /boot/grub/grub.cfg
```
### NON-UEFI
Install required package:
```bash
# pacman -S grub
```
Install GRUB:
```bash
# grub-install /dev/sda
```
Generate the /boot/grub/grub.cfg file:
```bash
# grub-mkconfig -o /boot/grub/grub.cfg
```
## Install a Desktop Environment
### GNOME
Install the Xorg display server:
```bash
# pacman -S xorg
```
Install the GNOME desktop environment:
```bash
# pacman -S gnome
```
Enable the GDM display manager and Network Manager:
```bash
# systemctl start gdm.service
# systemctl enable gdm.service
# systemctl enable NetworkManager.service
```
Exit from chroot:
```bash
# exit
```
Shutdown:
```bash
# shutdown now
```
Remove the live USB/medium and power back on.

### CINNAMON
Install the Xorg display server:
```bash
# pacman -S xorg
```
Install the Xorg terminal:
```bash
# pacman -S xterm
```
Install the Cinnamon desktop environment:
```bash
# pacman -S cinnamon
```
Install the GDM display manager:
```bash
# pacman -S gdm
```
Enable the GDM display manager and Network Manager:
```bash
# systemctl enable gdm.service
# systemctl enable NetworkManager.service
```
Exit from chroot:
```bash
# exit
```
Shutdown:
```bash
# shutdown now
```
Remove the live USB/medium and power back on.

### KDE
_**NOTE:** KDE doesn’t allow the root user to login directly. Create a new user and grant sudo rights._

Use the `useradd` command with the `-m` option to create a new user and the home directory for the new user:
```bash
# useradd -m deep
```
Set up user password:
```bash
# passwd deep
```
Install the `sudo` command:
```bash
# pacman -S sudo
```
_**NOTE:** The configuration file for sudo is /etc/sudoers.  This file hould always be edited with the visudo command.  The visudo command locks the sudoers file, saves edits to a temporary file, and then checks the file’s syntax before copying it to /etc/sudoers.)_

Set an editor for use when launching visudo:
```bash
# EDITOR=nano visudo
```
Add the following line for the newly created user:

> deep ALL=(ALL) ALL

Install the Xorg display server:
```bash
# pacman -S xorg
```
Install plasma, plasma-wayland-session, and kde-applications:
```bash
# pacman -S plasma plasma-wayland-session kde-applications 
```
Enable the SDDM display manager and Network Manager:
```bash
# systemctl enable sddm.service
# systemctl enable NetworkManager.service
```
Exit from chroot:
```bash
# exit
```
Shutdown:
```bash
# shutdown now
```
Remove the live USB/medium and power back on.
