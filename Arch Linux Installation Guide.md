![ArchLinux Logo](https://www.archlinux.org/static/logos/archlinux-logo-dark-90dpi.ebdee92a15b3.png)

# The Arch Linux Installation Guide
Deep Grewal - January 8, 2021

___
## Introduction
The installation of Arch has been and continues to be a rite-of-passage within the Linux community.  Although there are many guides that discuss the steps needed to install Arch Linux, I couldn't resist writing my very own guide.  This guide promises to be different by being a narrative-style approach to the topic.  The goals of this guide are to be educational, informative, and to make & keep it simple (in the spirit of the Arch philosophy).  The guide assumes that you have some proficiency with the command line and have a basic understanding of Linux.  I hope that this guide can help you succeed with this rite and put you in control of a system that you have built from the ground-up.

As an Arch user, the [Arch Wiki](https://wiki.archlinux.org/) will be an extremely valuable resource to you.  This resource is so well-composed and maintained that even non-Arch users glean knowledge, wisdom, and solutions from it daily.  

And now, may the adventure begin...
___
## Download the Arch ISO
The first thing that we need to do is to obtain an image of Arch Linux.  To do so, let's visit the the Arch Linux home page.

You can either click [here](https://www.archlinux.org/download) or visit the link below to download the latest ISO image file for Arch.

[https://www.archlinux.org/download](https://www.archlinux.org/download)
___
## Boot the System to the Arch ISO
Depending upon the type of system that Arch will be installed on, there are different methods of booting the Arch ISO.  

### Physical Machine
For physical machines, a bootable medium can be created from the Arch ISO file. 

#### Step 1: Prepare Live Bootable USB
Although an optical disk could have been used to create a bootable physical medium, USB was chosen due to its relevance.  The simplest way to prepare a live USB is via the `dd` command:

```
# dd if=/location/of/iso/file of=/device/entry/of/usb/drive
```

#### Step 2: Boot to USB
Insert the USB drive into the physical machine and boot into the USB drive.  BIOS settings/boot order may need to be adjusted to ensure that the physical machine boots from the USB drive.

#### Step 3: Confirm You are In
Once you have properly booted into the Arch ISO, a prompt similar to the one below will be displayed.  

```
root@archiso ~ #
```

### Virtual Machine
Virtual machines do not require a physical medium to be created.  The Arch ISO file can be mounted as a virtualized optical disk within the virtualized optical drive.

#### Confirm You are In
Once you have properly booted into the Arch ISO, a prompt similar to the one below will be displayed.

```
root@archiso ~ #
```
___
## Verify Connectivity to the Internet
During the installation, I prefer to have a wired connection to the Internet.  This guide has been written based on a machine that is connected to the Internet using a wired connection.

To check Internet connectivity, simply ping a website as shown in the example below.

```
root@archiso ~ # ping -c 4 archlinux.org
PING archlinux.org (95.217.163.246) 56(84) bytes of data.
64 bytes from archlinux.org (95.217.163.246): icmp_seq=1 ttl=47 time=206 ms
64 bytes from archlinux.org (95.217.163.246): icmp_seq=2 ttl=47 time=181 ms
64 bytes from archlinux.org (95.217.163.246): icmp_seq=3 ttl=47 time=181 ms
64 bytes from archlinux.org (95.217.163.246): icmp_seq=4 ttl=47 time=181 ms

--- archlinux.org ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3003ms
rtt min/avg/max/mdev = 180.629/187.159/205.907/10.828 ms
```
___
## Update the System Clock
Now that we have confirmed connectivity to the Internet, we can leverage NTP.  Use `timedatectl` command to ensure the system clock is accurate by enabling NTP.

``` 
root@archiso ~ # timedatectl set-ntp true
```
___
## Disk Partitioning
In order for us to have a working distribution, we have to create some partitions on the target hard disk so that we can properly install and run Arch Linux.  There are numerous tools and commands that can be used for disk partitioning.  This guide uses `fdisk` to create the partitions.  

The recommended partition schemes vary depending on whether or not the system has UEFI mode enabled.

So, let's verify if UEFI mode is enabled by checking for the existence of this directory:

```
root@archiso ~ # ls /sys/firmware/efi/efivars
ls: cannot access '/sys/firmware/efi/efivars': No such file or directory
```

Based on the output of the command above, we can determine that the system being used in this guide does **not** have UEFI.  

Now, we can list all existing disk and disk partitions.  For the purposes of this guide and for a simpler installation, a virtual machine has been created with a blank 20GB hard disk identified by `/dev/sda`.

```
root@archiso ~ # fdisk -l
Disk /dev/sda: 20 GiB, 21474836480 bytes, 41943040 sectors
Disk model: VBOX HARDDISK
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/loop0: 566.52 MiB, 594034688 bytes, 1160224 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
```

Let's begin the partitioning process on the `/dev/sda` hard disk.

```
root@archiso ~ # fdisk /dev/sda

Welcome to the fdisk (util-linux 2.36.1).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table.
Created a new DOS disklabel with disk identifier 0xa1936afe.

Command (m for help):
```

If you would like to see all `fdisk` commands, this is an ideal opporunity to press the `m` key and `Enter` to display a list of all commands.  You could do that for your information (FYI).  However, this guide will walk you through the commands that are needed to partition the disk.  

Before we create any partitions, let's review the recommended partition schemes.

### Recommended Partition Schemes
The schemes below are mere recommendations and can be altered to your liking.  Notably, there are many viewpoints on the size of a swap partition, so take these recommendations with a grain of salt and do what suits you best.  After all, that's the beauty of creating your very own system from the ground-up.

#### NON-UEFI
Mount Point | Partition | Partition Type | Partition Size | File System
------------|-----------|---------------|----------------|------------   
/mnt | /dev/sda1 | Linux | Remainder of the device | ext4 
[SWAP] |  /dev/sda2 | Linux swap | More than 512 MiB | ext4

<br/>

#### UEFI
Mount Point | Partition | Partition Type | Partition Size | File System
------------|-----------|---------------|----------------|------------    
/mnt/boot or mnt/efi | /dev/sda1 | EFI System Partition | 260MB - 512MB | fat32
/mnt | /dev/sda2 | Linux x86-64 root (/) | Remainder of the device | ext4
[SWAP] | /dev/sda3 | Linux swap |  More than 512MiB | ext4

### Create the Partitions
Since we have a non-UEFI system, it makes sense to follow the NON-UEFI partition scheme above.  This means that we will create 2 partitions: a swap partition (Linux swap) and a partition where root will be mounted (Linux).

We have a hard drive that is approximately 20GB in size, so we can easily spare approximately half of a gigabyte (512MB) for the Linux swap partition (`/dev/sda2`).  Leaving us with approximately 19.5GB for the Linux partition (`/dev/sda1`). 

#### Linux Partition
First, let's create the Linux partition (`/dev/sda1/`).

```
Command (m for help): n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): p   
Partition number (1-4, default 1): 1
First sector (2048-41943039, default 2048): 2048
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-41943049, default 41943039): +19.5G

Created a new partition of type 'Linux' and of size 19.5 GiB.
```

#### Swap Partition
Next, let's go ahead and create the Linux swap partition (`/dev/sda2/`).

```
Command (m for help): n
Partition type
   p   primary (1 primary, 0 extended, 3 free)
   e   extended (container for logical partitions)
Select (default p): p   
Partition number (2-4, default 2): 2
First sector (40896512-41943039, default 40896512): 40896512
Last sector, +/-sectors or +/-size{K,M,G,T,P} (40896512-41943039, default 41943039): 41943039

Created a new partition 2 of type 'Linux' and of size 511 MiB.
```

With the Linux swap partition, we need to change the partition type so that it is a true swap partition.  

If you would like to see all partition types, this is an ideal opporunity to press the `l` key and `Enter` to display a list of all partition types. You could do that for your information (FYI). However, this guide will walk you through the commands that are needed to partition the disk.

We now need to change the partition type of our intended Linux swap partition (`/dev/sda2`).

```
Command (m for help): t
Partition number (1,2, default 2): 2
Hex code or alias (type L to list all): 82

Changed type of partition 'Linux' to 'Linux swap / Solaris'.
```

#### Verify Proposed Partition Table
Before we save our changes and commit them to the disk, let's take a moment to verify that everything was done correctly.

```
Command (m for help): p
Disk /dev/sda: 20 GiB, 21474836480 bytes, 41943040 sectors
Disk model: VBOX HARDDISK
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xc12ff6e9

Device     Boot    Start      End  Sectors  Size Id Type
/dev/sda1           2048 40896511 40894464 19.5G 83 Linux
/dev/sda2       40896512 41943039  1046528  511M 82 Linux swap / Solaris
```

#### Write Partition Table to Disk
Finally, let's write the newly created partition table to the disk and exit the `fdisk` utility.

```
Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.

root@archiso ~ #
```
___
## Create File System
The creation of the partitions in the previous steps simply drew boundaries on the storage space offered by the hard disk and specified the type of space betweeen each boundry line.  In order for these partitions to be of any use, they should be initialized with a file system and have the swap partition enabled.

Again depending on the type of system (UEFI, non-UEFI), the process will vary.

### NON-UEFI
For our non-UEFI system, let's create an ext4 file system on the root partition (you may choose any other viable file system).

```
root@archiso ~ # mkfs.ext4 /dev/sda1
mke2fs 1.45.6 (20-Mar-2020)
Creating filesystem with 5111808 4k blocks and 1277952 inodes
Filesystem UUID: ca970c3e-5a47-468c-8ff7-6ba9dda277af
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208, 4096000

Allocating group tables: done
Writing inode tables: done
Creating journal (32768 blocks): done
Writing superblocks and filesystem accounting information: done
```

Also, let's prepare the swap partition.

```
root@archiso ~ # mkswap /dev/sda2
Setting up swap space version 1, size = 511 MiB (535818240 bytes)
no label, UUID=12e273d1-4dc4-4151-9ffe-6a09ae78be38

root@archiso ~ # swapon /dev/sda2
```

### UEFI
For the EFI partition type, create a FAT32 file system.

```
# mkfs.fat -F32 /dev/sda1
```

For the root partition, create an ext4 file system (or any other viable file system of your choice).

```
# mkfs.ext4 /dev/sda2
```

Prepare the swap partition:

```
# mkswap /dev/sda3
# swapon /dev/sda3
```
___
## Optimize Mirrors
Much like any other distro, Arch Linux relies on mirrors to obtain updates.  There are a plethora of mirrors that are hosted on hundreds of servers spanning the entire globe.  Usually, the mirrors which are geographically nearer in distance should provide for the fastest connection speeds.  

Arch Linux comes with a file known as the "mirrorlist" which contains all known mirrors.  However, this file is not optimized since it contains every single mirror.  We could manually go through this file and edit it, but that would take quite some time.  Luckily, there is a tool (Python script) called `reflector` that has been created that will automatically optimize the file for us.  We just need to provide some input.

### Sync the pacman Repository
Before we go about downloading any applications/tools, we should update the repository and ensure that we have the latest and greatest available to us.

```
root@archiso ~ # pacman -Syy
:: Synchronizing package databases...
 core                            132.8 KiB  5.64 MiB/s 00:00 [#################################] 100%
 extra                          1633.0 KiB  10.4 MiB/s 00:00 [#################################] 100%
 community                         5.3 MiB  8.91 MiB/s 00:00 [#################################] 100%
```

### Install reflector
With a fully-updated repository, we are in good-standing to install the `reflector` tool so that we can optimize the "mirrorlist" file for local mirrors.  

```
root@archiso ~ # pacman -S reflector

warning: reflector-2020.12.20.1-1 is up to date -- reinstalling
resolving dependencies...
looking for conflicting packages...

Packages (1) reflector-2020.12.20.1-1

Total Download Size:   0.03 MiB
Total Installed Size:  0.09 MiB
Net Upgrade Size:      0.00 MiB

:: Proceed with installation? [Y/n] Y
:: Retrieving packages...
 reflector-2020.12.20.1-1-any     25.9 KiB 0.00   B/s 00:00 [#################################] 100%
 (1/1) checking keys in keyring                             [#################################] 100%
 (1/1) checking package integrity                           [#################################] 100%
 (1/1) loading package files                                [#################################] 100%
 (1/1) checking for file conflicts                          [#################################] 100%
 (1/1) checking available disk space                        [#################################] 100%
 :: Processing package changes...
 (1/1) reinstalling reflector                               [#################################] 100%
 :: Running post-transaction hooks...
 (1/2) Reloading system manager configuration...
 (2/2) Arming ConditionNeedsUpdate... 
```
It seems that `reflector` was already up to date, but we went ahead and installed it again for the sake of this guide.

### Back up Existing Mirrorlist File
It doesn't hurt to make a backup of a file that is going to be changed.  Let's make backup of the "mirrorlist" file.

```
root@archiso ~ # cp /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.bak
```

### Run reflector Against the Mirrorlist File
Execute the `reflector` command to optimize the "mirrorlist" file.  The end result will be a leaner file which contains the most optimal entries.  Since I am in the United States, I have used "US" as the country code within the command.

```
root@archiso ~ # reflector -c "US" -f 12 -l 10 -n 12 --save /etc/pacman.d/mirrorlist
[2021-01-09 00:00:33] WARNING: failed to rate http(s) download (http://arch.mirror.square-r00t.net/community/os/x86_64/community.db): Download timed out after 5 second(s).
[2021-01-09 00:00:47] WARNING: failed to rate http(s) download (http://arch.mirror.constant.com/community/os/x86_64/community.db): Download timed out after 5 second(s).
```
___
## Install Arch Linux
Our system has now been prepared and optimized to take on the install of Arch Linux.  This phase involves mounting the Linux partition and installing desired packages using the `pacstrap` command on to the mounted Linux partition.

### Mount the Linux Partition
We must mount the root directory before we can perform any installation.

```
root@archiso ~ # mount /dev/sda1 /mnt
```

### Perform the Installation
Use the `pacstrap` command to install Arch Linux, required packages, and any additional packages (in this case, the nano text editor) to the mounted Linux partition.  Additional packages can always be installed later; the installation of `nano` as an additional package was included to demonstrate the ability of the `pacstrap` command (and to promote nano as my favorite text editor which will come in handy later in this guide).

```
root@archiso ~ # pacstrap /mnt base linux linux-firmware nano
```

After issuing the command above, the screen will be very busy with the installation of multiple packages which comprise the Arch Linux distribution.  Once complete, we can now say that we have installed Arch Linux!  But, there's still more to do: configuration, bootloader installation, and selecting a desktop environment.
___
## Configure the Installed Arch System
During the configuration phase, we will start things off by setting the root partition to mount automatically.  Then, we will set the timezone so that it reflects the current/local timezone.  Next, we will set the locale so that dates, times, numbers, etc. are formatted correctly based on the geographical locale of the machine. Also, we can take this opportunity to make some minor network configurations so that this machine has a proper and accurate identity on the network.  Finally, we can enhance the security of the machine by setting a password for the root user.

### Automating the Mounts
Let's create the `/etc/fstab` so that the root partition is mounted automatically when the system is booted.  This `/etc/fstab` file can be edited manually, but our goal is to simplify the installation of Arch Linux.  Similar to what we did with the `reflector` tool to automate the optimal mirror selection process, we will introduce and use yet another tool to create the `/etc/fstab` file.

We can automatically generate the fstab file using the `genfstab` command.

```
# genfstab -U /mnt >> /mnt/etc/fstab
```

### Change Root
Recall that we initially booted into this machine using an image file.  We are still in the root directory of the image file and our session is in RAM.  Now that we have installed Arch Linux, we need to switch to the physically installed root partition using the `arch-chroot` command.

Change root to the root directory at `/mnt`.

```
# arch-chroot /mnt
```

### Setting the Timezone

Use the `timedatectl` command to find your time zone:

```
# timedatectl list-timezones
```

Create a symbollic link to set the timezone (replace "America/Los_Angeles" with your timezone):

```
# ln -sf /usr/share/zoneinfo/America/Los_Angeles /etc/localtime
```

Run `hwclock` to generate /etc/adjtime:

```
# hwclock --systohc
```

### Setting Up Locale
_**NOTE:** Locale refers to language, number, date and currency formats  The file /etc/locale.gen contains locale settings and system languages and is commented by default._

Open the file and remove the "#" from the start of the line which contains your locale:
```
# nano /etc/locale.gen
```
This entry has been uncommented:
> en_US.UTF-8  

Generate the /etc/locale.conf file:
```
# locale-gen
# echo LANG=en_US.UTF-8  > /etc/locale.conf
# export LANG=en_US.UTF-8 
```
### Network Configuration
Create the /etc/hostname file and add the hostname entry:
```
# nano /etc/hostname
```
This entry has been added:
> ArchLinuxPC

Create the /etc/hosts file and add the proper entries:
```
# nano /etc/hosts
```
These entries have been added:
> 127.0.0.1	localhost\
> ::1		localhost\
> 127.0.1.1	ArchLinuxPC

### Root Password
Set up root passwd:
```
# passwd
```
___
## Install Grand Unified Bootloader (GRUB)
### UEFI
Install required packages:
```
# pacman -S grub efibootmgr
```
Create the directory where EFI partition will be mounted:
```
# mkdir /boot/efi
```
Mount the ESP partition:
```
# mount /dev/sda1 /boot/efi
```
Install GRUB:
```
# grub-install --target=x86_64-efi --bootloader-id=GRUB --efi-directory=/boot/efi
```
Generate the /boot/grub/grub.cfg file:
```
# grub-mkconfig -o /boot/grub/grub.cfg
```
### NON-UEFI
Install required package:
```
# pacman -S grub
```
Install GRUB:
```
# grub-install /dev/sda
```
Generate the /boot/grub/grub.cfg file:
```
# grub-mkconfig -o /boot/grub/grub.cfg
```
___
## Install a Desktop Environment

If you thought swap partition sizes and text editors were controversial, they don't compare to the tribalism that exists for desktop environments.  But that's what makes Linux amazing: the freedom to choose.  There are many desktop environments that can be used with Arch Linux.  The most popular (at the time of this writing) have been included below. 

Choose your desktop environment, perform the installation, and finalize the configuration.  Then, let's reconnect in the Summary section of this tutorial.

### GNOME
Install the Xorg display server.

```
# pacman -S xorg
```

Install the GNOME desktop environment.

```
# pacman -S gnome
```

Enable the GDM display manager and Network Manager.

```
# systemctl start gdm.service
# systemctl enable gdm.service
# systemctl enable NetworkManager.service
```

Exit from chroot.

```
# exit
```

Shutdown.

```
# shutdown now
```

Remove the live USB/medium and power back on.

### CINNAMON

Install the Xorg display server.

```
# pacman -S xorg
```

Install the Xorg terminal.

```
# pacman -S xterm
```

Install the Cinnamon desktop environment.

```
# pacman -S cinnamon
```

Install the GDM display manager.

```
# pacman -S gdm
```

Enable the GDM display manager and Network Manager.

```
# systemctl enable gdm.service
# systemctl enable NetworkManager.service
```

Exit from chroot.

```
# exit
```

Shutdown.

```
# shutdown now
```

Remove the live USB/medium and power back on.

### KDE
_**NOTE:** KDE doesn’t allow the root user to login directly. Create a new user and grant sudo rights._

Use the `useradd` command with the `-m` option to create a new user and the home directory for the new user.

```
# useradd -m deep
```

Set up user password.

```
# passwd deep
```

Install the `sudo` command.

```
# pacman -S sudo
```

_**NOTE:** The configuration file for `sudo` is `/etc/sudoers`.  This file should always be edited with the `visudo` command.  The `visudo` command locks the "sudoers" file, saves edits to a temporary file, and then checks the file’s syntax before copying it to `/etc/sudoers`.)_

Set an editor for use when launching `visudo`.

```
# EDITOR=nano visudo
```

Add the following line for the newly created user.

> deep ALL=(ALL) ALL

Install the Xorg display server.

```
# pacman -S xorg
```

Install `plasma`, `plasma-wayland-session`, and `kde-applications`.

```
# pacman -S plasma plasma-wayland-session kde-applications 
```

Enable the SDDM display manager and Network Manager.

```
# systemctl enable sddm.service
# systemctl enable NetworkManager.service
```

Exit from chroot.

```
# exit
```

Shutdown.

```
# shutdown now
```

Remove the live USB/medium and power back on.
___
## Summary
Congratulations! You now have a working Arch Linux system which you have designed based on your choices and preferences.  The journey of a thousand miles has just begun with one step.  There is much more to install, configure, tweak, and learn.  I hope that you have enjoyed this guide and have gained some insight into the installation of Arch Linux.  You can now boast to your friends and colleagues about your distro of choice.  Don't hesitate to sprinkle in the occasional, "By the way, I use Arch" in casual conversations.  For more information 