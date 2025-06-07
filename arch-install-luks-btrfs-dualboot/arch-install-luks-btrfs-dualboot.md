---
title: arch-install-luks-btrfs-dualboot
tags:
  - Notebooks/Geek_Room
created: 2024-04-21 19:12
modified: 2024-04-22 14:29
moc: "[[_geek-room]]"
---
<!--ts-->
* [arch-install-luks-btrfs-dualboot](#arch-install-luks-btrfs-dualboot)
   * [Source Links](#source-links)
* [Arch Linux Full-Disk Encryption Base System Installation Guide](#arch-linux-full-disk-encryption-base-system-installation-guide)
   * [Preface](#preface)
   * [Pre-installation](#pre-installation)
      * [Connect to the internet](#connect-to-the-internet)
         * [Wifi Settings](#wifi-settings)
      * [Update the system clock](#update-the-system-clock)
      * [Optionally (recommended) update mirrorlist](#optionally-recommended-update-mirrorlist)
      * [Arch install by SSH](#arch-install-by-ssh)
      * [Virtualbox tty resolution settings](#virtualbox-tty-resolution-settings)
      * [Preparing the disk](#preparing-the-disk)
         * [Update btrfs-progs](#update-btrfs-progs)
         * [Display disks and partitions](#display-disks-and-partitions)
         * [Create EFI System and Linux LUKS partitions](#create-efi-system-and-linux-luks-partitions)
         * [Create the LUKS1 encrypted container on the Linux LUKS partition (GRUB does not support LUKS2 as of May 2019)](#create-the-luks1-encrypted-container-on-the-linux-luks-partition-grub-does-not-support-luks2-as-of-may-2019)
         * [Open the container (decrypt it and make available at /dev/mapper/cryptbtrfs)](#open-the-container-decrypt-it-and-make-available-at-devmappercryptbtrfs)
      * [Preparing BTRFS Volume/Subvolumes](#preparing-btrfs-volumesubvolumes)
         * [Formatting as btrfs now when it is already encrypted](#formatting-as-btrfs-now-when-it-is-already-encrypted)
         * [Mount filesystem](#mount-filesystem)
         * [Create subvolumes](#create-subvolumes)
         * [Disable copy on write on /var/log and /tmp](#disable-copy-on-write-on-varlog-and-tmp)
         * [Mount BTRFS Subvolumes](#mount-btrfs-subvolumes)
      * [Preparing the EFI partition](#preparing-the-efi-partition)
         * [Create FAT32 filesystem on the EFI system partition](#create-fat32-filesystem-on-the-efi-system-partition)
         * [Create mountpoint for EFI system partition at /efi for compatibility with grub-install and mount it](#create-mountpoint-for-efi-system-partition-at-efi-for-compatibility-with-grub-install-and-mount-it)
   * [Arch Base Installation](#arch-base-installation)
      * [Install necessary packages](#install-necessary-packages)
   * [Configure the system](#configure-the-system)
      * [Generate an fstab file](#generate-an-fstab-file)
      * [Enter new system chroot](#enter-new-system-chroot)
         * [TODO: (Replace it) At this point you should have the following partitions and logical volumes:](#todo-replace-it-at-this-point-you-should-have-the-following-partitions-and-logical-volumes)
      * [Time zone](#time-zone)
         * [Set the time zone](#set-the-time-zone)
         * [Run hwclock to generate /etc/adjtime](#run-hwclock-to-generate-etcadjtime)
      * [Localization](#localization)
         * [Uncomment en_US.UTF-8 UTF-8 in /etc/locale.gen and generate locale](#uncomment-en_usutf-8-utf-8-in-etclocalegen-and-generate-locale)
         * [Create locale.conf and set the LANG variable](#create-localeconf-and-set-the-lang-variable)
      * [Network configuration](#network-configuration)
         * [Create the hostname file](#create-the-hostname-file)
         * [Add matching entries to hosts](#add-matching-entries-to-hosts)
      * [(Optional) Swap File Creation](#optional-swap-file-creation)
      * [Initramfs](#initramfs)
         * [Add the keyboard, encrypt, and btrfs hooks to /etc/mkinitcpio.conf](#add-the-keyboard-encrypt-and-btrfs-hooks-to-etcmkinitcpioconf)
         * [Recreate the initramfs image](#recreate-the-initramfs-image)
      * [Fundamental Software Installation](#fundamental-software-installation)
         * [Graphical Drivers and Xorg](#graphical-drivers-and-xorg)
            * [AMD Cards](#amd-cards)
            * [Nvidia Cards](#nvidia-cards)
            * [Virtualbox Support](#virtualbox-support)
      * [Root password](#root-password)
         * [Set the root password](#set-the-root-password)
      * [Add Linux User](#add-linux-user)
      * [Boot loader](#boot-loader)
         * [Install GRUB](#install-grub)
         * [Configure GRUB to allow booting from /boot on a LUKS1 encrypted partition](#configure-grub-to-allow-booting-from-boot-on-a-luks1-encrypted-partition)
         * [Set kernel parameter to unlock the BTRFS physical volume at boot using encrypt hook](#set-kernel-parameter-to-unlock-the-btrfs-physical-volume-at-boot-using-encrypt-hook)
            * [UUID is the partition containing the LUKS container](#uuid-is-the-partition-containing-the-luks-container)
         * [Install GRUB to the mounted ESP for UEFI booting](#install-grub-to-the-mounted-esp-for-uefi-booting)
         * [Generate GRUB's configuration file](#generate-grubs-configuration-file)
      * [(recommended) Embed a keyfile in initramfs](#recommended-embed-a-keyfile-in-initramfs)
         * [Create a keyfile and add it as LUKS key](#create-a-keyfile-and-add-it-as-luks-key)
         * [Add the keyfile to the initramfs image](#add-the-keyfile-to-the-initramfs-image)
         * [Recreate the initramfs image](#recreate-the-initramfs-image-1)
         * [Set kernel parameters to unlock the LUKS partition with the keyfile using encrypt hook](#set-kernel-parameters-to-unlock-the-luks-partition-with-the-keyfile-using-encrypt-hook)
         * [Regenerate GRUB's configuration file](#regenerate-grubs-configuration-file)
         * [Restrict /boot permissions](#restrict-boot-permissions)
   * [Post-installation](#post-installation)
   * [Windows 11 Installation](#windows-11-installation)
      * [Speeding up LUKS decryption in GRUB](#speeding-up-luks-decryption-in-grub)
      * [Docker on BTRFS Storage Driver](#docker-on-btrfs-storage-driver)

<!-- Created by https://github.com/ekalinin/github-markdown-toc -->
<!-- Added by: box, at: Sat Jun  7 10:17:37 AM CEST 2025 -->

<!--te-->

# arch-install-luks-btrfs-dualboot

Arch Installation includes LUKS, BTRFS+Backup, Base Installation, Dualboot with Win11.

## Source Links

* **MAIN ARTICLE:** https://gist.github.com/huntrar/e42aee630bee3295b2c671d098c81268
* https://www.nishantnadkarni.tech/posts/arch_installation/
* https://www.dwarmstrong.org/archlinux-install/
* https://btrfs.readthedocs.io/en/latest/btrfs-man5.html#compression
* https://github.com/Szwendacz99/Arch-install-encrypted-btrfs
* https://gist.github.com/mruiz42/83d9a232e7592d65d953671409a2aab9
* https://wiki.archlinux.org/title/btrfs
* https://wiki.archlinux.org/title/dm-crypt/Encrypting_an_entire_system
* https://wiki.tnonline.net/w/Btrfs/Compression

# Arch Linux Full-Disk Encryption Base System Installation Guide 
This guide provides instructions for an Arch Linux installation featuring full-disk encryption via BTRFS on LUKS and an encrypted boot partition (GRUB) for UEFI systems.

Following the main installation are further instructions to harden against Evil Maid attacks via UEFI Secure Boot custom key enrollment and self-signed kernel and bootloader.

## Preface
You will find most of this information pulled from the [Arch Wiki](https://wiki.archlinux.org/index.php/Dm-crypt/Encrypting_an_entire_system#Encrypted_boot_partition_(GRUB)) and other resources linked thereof.

*Note:* The system was installed on an NVMe SSD, substitute `/dev/nvme0nX` with `/dev/sdX` or your device as needed.

## Pre-installation
### Connect to the internet
Plug in your Ethernet and go, or for wireless consult the all-knowing [Arch Wiki](https://wiki.archlinux.org/index.php/installation_guide#Connect_to_the_internet).

#### Wifi Settings
```shell
iwctl
[iwd]# device list -> note your device
[iwd]# station <device> get-networks
[iwd]# station <device> connect <SSID>

# second variant
iwctl --passphrase <heslo> station <device> connect <SSID>

# test connection
ping archlinux.org
```

### Update the system clock
```shell
timedatectl set-ntp true
```

### Optionally (recommended) update mirrorlist
```shell
pacman -Syy
pacman -S reflector
cp /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.bak

reflector --country 'Czechia' --age 24 --verbose --sort rate --save /etc/pacman.d/mirrorlist
# OR
reflector -c "CZ" -f 12 -l 10 -n 12 --save /etc/pacman.d/mirrorlist
```

### Arch install by SSH

```shell
# in installed station
# create/change password for root, ssh is set correctly in arch-iso
passwd
# remember ip address of computer
ip a
# remote station for installation process
# start ssh session with created password for root
ssh root@[ip-address]
```

### Virtualbox tty resolution settings
```shell
pacman -S fbset terminus-font
fbset -g 2048 1080 2048 1080 32
setfont ter-132n
```

### Preparing the disk
#### Update btrfs-progs
```shell
pacman -Syy btrfs-progs
```
#### Display disks and partitions
```shell
lsblk
```
#### Create EFI System and Linux LUKS partitions

| Number | Start (sector) | End (sector) | Size      | Code | Name       |
| ------ | -------------- | ------------ | --------- | ---- | ---------- |
| 1      | 2048           | 1130495      | 256.0 MiB | EF00 | EFI System |
| 2      | 1130496        | 976773134    | 465.2 GiB | 8309 | Linux LUKS |

```shell
# includes the luks partition 8309 ID
gdisk /dev/nvme0n1

# alternative apps - cannot be used for luks1
cfdisk
fdisk
```

```
# we keep some empty space for windows after linux partition
o
n
[Enter]
0
+256M
ef00
n
[Enter]
+256G  # for example 1/2 disk space
[Enter]
8309
w
```

#### Create the LUKS1 encrypted container on the Linux LUKS partition (GRUB does not support LUKS2 as of May 2019)
```shell
cryptsetup luksFormat --type luks1 --use-random -S 1 -s 512 -h sha512 -i 1000 /dev/nvme0n1p2
```

#### Open the container (decrypt it and make available at /dev/mapper/cryptbtrfs)
```shell
cryptsetup open /dev/nvme0n1p2 cryptbtrfs
```

### Preparing BTRFS Volume/Subvolumes
#### Formatting as btrfs now when it is already encrypted
```shell
mkfs.btrfs -L "Arch Linux" /dev/mapper/cryptbtrfs
```

#### Mount filesystem
```shell
mount /dev/mapper/cryptbtrfs /mnt
```

#### Create subvolumes

This scheme can be adjusted to your needs, I'd suggest at least one subvolume for root (@) and one for snapshots (.@snapshots). varlog and tmp are created to easily disable Copy on Write on /var/log and /tmp.

```shell
btrfs sub cr /mnt/@
btrfs sub cr /mnt/@home
btrfs sub cr /mnt/@tmp
btrfs sub cr /mnt/@log
btrfs sub cr /mnt/@pkg
btrfs sub cr /mnt/@docker
btrfs sub cr /mnt/.@snapshots
```

#### Disable copy on write on /var/log and /tmp

```shell
chattr +C /mnt/@log
chattr +C /mnt/@tmp  
umount /mnt
```

#### Mount BTRFS Subvolumes

```shell
mount -o defaults,noatime,discard,ssd,compress=zstd,subvol=@ /dev/mapper/cryptbtrfs /mnt  

mkdir -p /mnt/{home,var/log,var/cache/pacman/pkg,var/lib/docker,tmp,.snapshots}

mount -o defaults,noatime,discard,ssd,compress=zstd,subvol=@home /dev/mapper/cryptbtrfs /mnt/home && \
mount -o defaults,noatime,discard,ssd,compress=zstd,subvol=@tmp /dev/mapper/cryptbtrfs /mnt/tmp && \
mount -o defaults,noatime,discard,ssd,compress=zstd,subvol=@log /dev/mapper/cryptbtrfs /mnt/var/log && \
mount -o defaults,noatime,discard,ssd,compress=zstd,subvol=@pkg /dev/mapper/cryptbtrfs /mnt/var/cache/pacman/pkg && \
mount -o defaults,noatime,discard,ssd,compress=zstd,subvol=@docker /dev/mapper/cryptbtrfs /mnt/var/lib/docker && \
mount -o defaults,noatime,discard,ssd,compress=zstd,subvol=.@snapshots /dev/mapper/cryptbtrfs /mnt/.snapshots
```

### Preparing the EFI partition
#### Create FAT32 filesystem on the EFI system partition
```shell
mkfs.fat -F32 /dev/nvme0n1p1
```

#### Create mountpoint for EFI system partition at /efi for compatibility with grub-install and mount it
```shell
mkdir /mnt/efi
mount /dev/nvme0n1p1 /mnt/efi
```

## Arch Base Installation
### Install necessary packages
```shell
pacstrap /mnt/ base base-devel linux linux-headers linux-firmware polkit git btrfs-progs efibootmgr mkinitcpio dhcpcd bash-completion sudo neovim nano
```

## Configure the system
### Generate an fstab file
```shell
genfstab -U /mnt >> /mnt/etc/fstab
```
Check if the `fstab` is correctly created with defined btrfs parameters from previous chapter with neovim.

### Enter new system chroot
```shell
arch-chroot /mnt
```

#### TODO: (Replace it) At this point you should have the following partitions and logical volumes:
```shell
lsblk
```

| NAME           | MAJ:MIN | RM | SIZE   | RO | TYPE  | MOUNTPOINT            |
| -------------- | ------- | -- | ------ | -- | ----- | --------------------- |
| nvme0n1        | 259:0   | 0  | 465.8G | 0  | disk  |                       |
| ├─nvme0n1p1    | 259:5   | 0  | 256M   | 0  | part  | /efi                  |
| ├─nvme0n1p2    | 259:6   | 0  | 465.2G | 0  | part  |                       |
| ..└─cryptbtrfs | 254:0   | 0  | 465.2G | 0  | crypt | /.snapshots           |
|                |         |    |        |    |       | /var/lib/docker       |
|                |         |    |        |    |       | /var/cache/pacman/pkg |
|                |         |    |        |    |       | /var/log              |
|                |         |    |        |    |       | /tmp                  |
|                |         |    |        |    |       | /home                 |
|                |         |    |        |    |       | /                     |

### Time zone
#### Set the time zone
Replace `Europe/Prague` with your respective timezone found in `/usr/share/zoneinfo`
```shell
ln -sf /usr/share/zoneinfo/Europe/Prague /etc/localtime
```

#### Run `hwclock` to generate ```/etc/adjtime```
Assumes hardware clock is set to UTC
```shell
hwclock --systohc --utc
```

### Localization
#### Uncomment ```en_US.UTF-8 UTF-8``` in ```/etc/locale.gen``` and generate locale
```shell
locale-gen
```

#### Create ```locale.conf``` and set the ```LANG``` variable
```shell
echo LANG=en_US.UTF-8 > /etc/locale.conf
export LANG=en_US.UTF-8
```

### Network configuration
#### Create the hostname file
```shell
echo myhostname > /etc/hostname
```

This is a unique name for identifying your machine on a network.

#### Add matching entries to hosts
```shell
nvim /etc/hosts
```
```shell
127.0.0.1 localhost
::1 localhost
127.0.1.1 myhostname
```

### (Optional) Swap File Creation

Now create empty (with 0 size) swap file:
Create separate subvolume for swapfile. This subvolume is needed to let you make snapshot of /, which would not be possible with any file in it with CoW disabled!

```shell
btrfs su create /swap
chattr +C /swap

# Copy on Write should always be disabled on swap file, so it will be done in the next step
touch /swap/swapfile

# Check if C attribute is enabled (should be already if created in folder with disabled CoW attribute)
lsattr /swap/swapfile

# If not then disable CoW for swapfile manually
chattr +C /swap/swapfile  

# Expanding empty file to 4GiB swap file
dd if=/dev/zero of=/swap/swapfile bs=1024K count=4096  
chmod 600 /swap/swapfile

# Format the swap file.
mkswap /swap/swapfile

# Turn swap file on.
swapon /swap/swapfile  

# You also need to update /etc/fstab to mount swapfile on boot
/swap/swapfile none swap sw 0 0
```

### Initramfs
#### Add the ```keyboard```, ```encrypt```, and ```btrfs``` hooks to ```/etc/mkinitcpio.conf```
*Note:* ordering matters.
```shell
HOOKS=(base udev autodetect modconf kms keyboard keymap consolefont block encrypt btrfs filesystems fsck)

# Add btrfsck to binaries
BINARIES=(btrfsck)
```

#### Recreate the initramfs image
```shell
mkinitcpio -P
```

### Fundamental Software Installation
```shell
pacman -S openssh networkmanager wpa_supplicant netctl
systemctl enable NetworkManager
systemctl enable sshd

pacman -S amd-ucode (for AMD) pacman -S intel-ucode (for INTEL)
mkinitcpio -p linux
```

#### Graphical Drivers and Xorg
##### AMD Cards
```shell
pacman -S xorg
pacman -S mesa
```
##### Nvidia Cards
```shell
pacman -S xorg
pacman -S nvidia nvidia-utils

sudo vim /etc/mkinitcpio.conf
# edit Modules and Files
MODULES=(nvidia nvidia_modeset nvidia_uvm nvidia_drm ...)
FILES="/etc/modprobe.d/nvidia.conf"

sudo mkinitcpio -P
nvim /etc/modprobe.d/nvidia.conf
# Add row to the file
options nvidia_drm modeset=1
```

##### Virtualbox Support
```shell
pacman -S virtualbox-guest-utils xf86-video-vmware
```

### Root password
#### Set the root password
```shell
passwd
```

### Add Linux User
```shell
useradd -m -g users -G wheel,storage,power -s /bin/bash <user>
passwd <user>
pacman -S sudo
EDITOR=nvim visudo,  (uncomment) %wheel ALL=(ALL:ALL) ALL
```

### Boot loader
#### Install GRUB
```shell
pacman -S grub efibootmgr os-prober dosfstools mtools
```

#### Configure GRUB to allow booting from /boot on a LUKS1 encrypted partition
```shell
nvim /etc/default/grub
```
```
GRUB_ENABLE_CRYPTODISK=y
```

#### Set kernel parameter to unlock the BTRFS physical volume at boot using ```encrypt``` hook
##### UUID is the partition containing the LUKS container
```shell
blkid
```
```shell
/dev/nvme0n1p2: UUID="xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx" TYPE="crypto_LUKS" PARTLABEL="Linux LUKS" PARTUUID="xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
```
```shell
/etc/default/grub
```
```shell
# allow-discards is only for ssd to let trim work with encryption enabled
GRUB_CMDLINE_LINUX="... cryptdevice=UUID=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx:cryptbtrfs:allow-discards"
```

#### Install GRUB to the mounted ESP for UEFI booting
```shell
grub-install --target=x86_64-efi --bootloader-id=ARCH --efi-directory=/efi --recheck
```

#### Generate GRUB's configuration file
```
grub-mkconfig -o /boot/grub/grub.cfg
```

### (recommended) Embed a keyfile in initramfs

This is done to avoid having to enter the encryption passphrase twice (once for GRUB, once for initramfs.)

#### Create a keyfile and add it as LUKS key
```shell
mkdir /root/secrets && chmod 700 /root/secrets
head -c 64 /dev/urandom > /root/secrets/crypto_keyfile.bin && chmod 600 /root/secrets/crypto_keyfile.bin
cryptsetup -v luksAddKey -i 1 /dev/nvme0n1p2 /root/secrets/crypto_keyfile.bin
```

#### Add the keyfile to the initramfs image
```shell
nvim /etc/mkinitcpio.conf
```
```shell
FILES=(/root/secrets/crypto_keyfile.bin)
```

#### Recreate the initramfs image
```shell
mkinitcpio -P
```

#### Set kernel parameters to unlock the LUKS partition with the keyfile using ```encrypt``` hook
```shell
/etc/default/grub
```
```shell
GRUB_CMDLINE_LINUX="... cryptkey=rootfs:/root/secrets/crypto_keyfile.bin"
```

#### Regenerate GRUB's configuration file
```shell
grub-mkconfig -o /boot/grub/grub.cfg
```

#### Restrict ```/boot``` permissions
```shell
chmod 700 /boot
```

The installation is now complete. Exit the chroot and reboot.
```shell
exit
reboot
```

## Post-installation
Your system should now be fully installed, bootable, and fully encrypted.

If you embedded the keyfile in the initramfs image, it should only require your encryption passphrase once to unlock to the system.

For the standard Arch Linux post-installation steps, [RTFM](https://wiki.archlinux.org/index.php/General_recommendations).

## Windows 11 Installation

We have prepared an empty partition right after `nvme0n1p2` on Windows 11. Now you can proceed with the installation. After rebooting, insert a USB stick with Windows 11, select the EFI USB stick from the computer's boot menu, and start the Windows 11 installation.

You must choose an `advanced custom installation`, where in the disk editing window you select the appropriate empty area. You do not need to format the area, just click on it and continue; Windows will automatically divide it into its required partitions and begin a complete installation of Windows. Continue normally through the entire installation.

The current version of Windows 11 will correctly insert its EFI into the ESP partition we created with Arch, however, it will save itself as the primary owner first in the GPT table. Sometimes it can even happen that your Arch EFI record from the GPT table gets deleted. After installing Windows, we now need to **boot back into Arch** using the Arch USB stick.

Now that we have booted from the arch iso, we can repeat the steps for `arch-chroot` to get into our system. This procedure can always be used in case of any problems to rescue your installation.

```shell
# first, open encrypted disk with your master password
cryptsetup open /dev/nvme0n1p2 cryptbtrfs
# mount your encrypted partition
mount /dev/mapper/cryptbtrfs /mnt
# mount all
mount -o defaults,noatime,discard,ssd,compress=zstd,subvol=@ /dev/mapper/cryptbtrfs /mnt  

mount -o defaults,noatime,discard,ssd,compress=zstd,subvol=@home /dev/mapper/cryptbtrfs /mnt/home && \
mount -o defaults,noatime,discard,ssd,compress=zstd,subvol=@tmp /dev/mapper/cryptbtrfs /mnt/tmp && \
mount -o defaults,noatime,discard,ssd,compress=zstd,subvol=@log /dev/mapper/cryptbtrfs /mnt/var/log && \
mount -o defaults,noatime,discard,ssd,compress=zstd,subvol=@pkg /dev/mapper/cryptbtrfs /mnt/var/cache/pacman/pkg && \
mount -o defaults,noatime,discard,ssd,compress=zstd,subvol=@docker /dev/mapper/cryptbtrfs /mnt/var/lib/docker && \
mount -o defaults,noatime,discard,ssd,compress=zstd,subvol=.@snapshots /dev/mapper/cryptbtrfs /mnt/.snapshots
# now, arch-chroot to our system
arch-chroot /mnt
# install ntfs support 
sudo pacman -S ntfs-3g
# uncomment os-prober in the /etc/default/grub
GRUB_DISABLE_OS_PROBER=false
# regenerate grub.conf and os-prober will find win11 parition
grub-mkconfig -o /boot/grub/grub.cfg
# check that the windows is found, if not, try to mount ntfs disk and run mkconfig again
```

Now we have correctly set up GRUB and it would suffice to restart the computer and the BIOS will correctly choose the last written EFI, which is our Grub2 with Arch. However, some of today's BIOS are very 'flawed'. Usually, they take the first record where Windows is, or EVEN delete other records from the GPT table. They don't support multiple records for EFIs. Unfortunately, this is the reality. However, we will manage this and before resetting, we will modify the GPT ourselves.

```shell
# list actual state of gpt
efibootmgr -v
# change boot order to START EFI with Arch Grub!
# use your IDs in your table
# 0001 includes Arch with grub2
efibootmgr -o 0001,0000,0012,0013,0014,0015,0016,0017,0018,0019
```

**Congratulations! You have successfully set up a dual-boot system with ARCH LINUX on the same disk.** 

Just restart, and importantly, do not turn on BIOS and its boot order! Hooray, Arch with Grub loads beautifully.

It must be noted that the ESP part of EFI for Arch contains only a small binary file, everything else is not in EFI! We cleverly stored this in `/boot` with grub and kernel images, while `/efi` contains only then a binary file. This way, we can beautifully use only a small space in ESP for our purposes and any number of large kernel files in `/boot` in a secured part of the disk.

### Speeding up LUKS decryption in GRUB

**Warning:** Before following this section, make sure you understand the importance of high entropy passwords. Information on how to generate secure passwords can be found at Wikipedia:Password strength.

Upon boot GRUB may in some cases take a long time to verify the password. This can be due to a high PBKDF iteration count, which you can check as follows:

```shell
cryptsetup luksDump /dev/nvme0n1p2
```

The problem is that the iteration count for a given `keyslot` is generated when the key is added to ensure a balance between being high enough to protect against brute force attacks and low enough to allow for fast key derivation by estimating the capabilities of your computer. However, when GRUB is started, it might **not have** the same computational resources at hand, thus being **vastly slower**.

If your **password provides enough entropy** to counter common attacks by itself, you can lower this number:

```shell
cryptsetup luksChangeKey --pbkdf-force-iterations 1000 /dev/nvme0n1p2
```

A minimum of 1000 iterations is recommended as per RFC 2898, but you should aim for higher values if you can (The cost for an attacker as well as the time for key derivation scale linearly).

**Tip:** GRUB tries enabled key slots sequentially. When adding keys, the `--key-slot` option can be used to specify a key slot explicitly.

### Docker on BTRFS Storage Driver

[Docker on Arch](https://wiki.archlinux.org/title/docker).

BTRFS is ideal for docker snapshots. Follow settings by this [original article](https://docs.docker.com/storage/storagedriver/btrfs-driver/).

