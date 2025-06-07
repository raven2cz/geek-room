---
title: arch-install-luks-btrfs
tags:
  - Notebooks/Geek_Room
created: 2024-10-14 08:49
modified: 2023-11-23T18:06:53.440Z
moc: "[[_geek-room]]"
---
```table-of-contents
title: 
style: nestedList # TOC style (nestedList|inlineFirstLevel)
minLevel: 0 # Include headings from the specified level
maxLevel: 3 # Include headings up to the specified level
includeLinks: true # Make headings clickable
debugInConsole: false # Print debug info in Obsidian console
```

# arch-install-luks-btrfs

Arch Installation includes LUKS, BTRFS+Backup, Base Installation, Secure Boot.

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
gdisk /dev/nvme0n1

# alternative apps
cfdisk
fdisk
```

```
o
n
[Enter]
0
+256M
ef00
n
[Enter]
[Enter]
[Enter]
8309
w
```

#### Create the LUKS1 encrypted container on the Linux LUKS partition (GRUB does not support LUKS2 as of May 2019)
```shell
cryptsetup luksFormat --type luks1 --use-random -S 1 -s 512 -h sha512 -i 5000 /dev/nvme0n1p2
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
mount -o defaults,noatime,discard,ssd,subvol=@ /dev/mapper/cryptbtrfs /mnt  
mkdir -p /mnt/{home,var/log,var/cache/pacman/pkg,var/lib/docker,tmp,.snapshots}

# Discard and ssd options and are for ssd disks only
mount -o defaults,noatime,discard,ssd,subvol=@home /dev/mapper/cryptbtrfs /mnt/home
mount -o defaults,noatime,discard,ssd,subvol=@tmp /dev/mapper/cryptbtrfs /mnt/tmp
mount -o defaults,noatime,discard,ssd,subvol=@log /dev/mapper/cryptbtrfs /mnt/var/log
mount -o defaults,noatime,discard,ssd,subvol=@pkg /dev/mapper/cryptbtrfs /mnt/var/cache/pacman/pkg/
mount -o defaults,noatime,discard,ssd,subvol=@docker /dev/mapper/cryptbtrfs /mnt/var/lib/docker
mount -o defaults,noatime,discard,ssd,subvol=.@snapshots /dev/mapper/cryptbtrfs /mnt/.snapshots
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

### (recommended) Secure Boot - Hardening against Evil Maid attacks
With an encrypted boot partition, nobody can see or modify your kernel image or initramfs, but you would be still vulnerable to [Evil Maid](https://www.schneier.com/blog/archives/2009/10/evil_maid_attac.html) attacks.

One possible solution is to use UEFI Secure Boot. Get rid of preloaded Secure Boot keys (you really don't want to trust Microsoft and OEM), enroll [your own Secure Boot keys](https://wiki.archlinux.org/index.php/Secure_Boot#Using_your_own_keys) and sign the GRUB boot loader with your keys. Evil Maid would be unable to boot modified boot loader (not signed by your keys) and the attack is prevented.

#### Creating keys
The following steps should be performed as the `root` user, with accompanying files stored in the `/root` directory.

##### Install `efitools`
```shell
pacman -S efitools
```

##### Create a GUID for owner identification
```shell
uuidgen --random > GUID.txt
```

##### Platform key
CN is a Common Name, which can be written as anything.

```shell
openssl req -newkey rsa:4096 -nodes -keyout PK.key -new -x509 -sha256 -days 3650 -subj "/CN=my Platform Key/" -out PK.crt
openssl x509 -outform DER -in PK.crt -out PK.cer
cert-to-efi-sig-list -g "$(< GUID.txt)" PK.crt PK.esl
sign-efi-sig-list -g "$(< GUID.txt)" -k PK.key -c PK.crt PK PK.esl PK.auth
```

##### Sign an empty file to allow removing Platform Key when in "User Mode"
```shell
sign-efi-sig-list -g "$(< GUID.txt)" -c PK.crt -k PK.key PK /dev/null rm_PK.auth
```

##### Key Exchange Key
```shell
openssl req -newkey rsa:4096 -nodes -keyout KEK.key -new -x509 -sha256 -days 3650 -subj "/CN=my Key Exchange Key/" -out KEK.crt
openssl x509 -outform DER -in KEK.crt -out KEK.cer
cert-to-efi-sig-list -g "$(< GUID.txt)" KEK.crt KEK.esl
sign-efi-sig-list -g "$(< GUID.txt)" -k PK.key -c PK.crt KEK KEK.esl KEK.auth
```

##### Signature Database key
```shell
openssl req -newkey rsa:4096 -nodes -keyout db.key -new -x509 -sha256 -days 3650 -subj "/CN=my Signature Database key/" -out db.crt
openssl x509 -outform DER -in db.crt -out db.cer
cert-to-efi-sig-list -g "$(< GUID.txt)" db.crt db.esl
sign-efi-sig-list -g "$(< GUID.txt)" -k KEK.key -c KEK.crt db db.esl db.auth
```

#### Signing bootloader and kernel
When Secure Boot is active (i.e. in "User Mode") you will only be able to launch signed binaries, so you need to sign your kernel and boot loader.

Install `sbsigntools`
```shell
pacman -S sbsigntools
```
```shell
sbsign --key db.key --cert db.crt --output /boot/vmlinuz-linux /boot/vmlinuz-linux
sbsign --key db.key --cert db.crt --output /efi/EFI/arch/grubx64.efi /efi/EFI/arch/grubx64.efi
```

##### Automatically sign bootloader and kernel on install and updates
It is necessary to sign GRUB with your UEFI Secure Boot keys every time the system is updated via `pacman`. This can be accomplished with a [pacman hook](https://jlk.fjfi.cvut.cz/arch/manpages/man/alpm-hooks.5).

Create the hooks directory
```shell
mkdir -p /etc/pacman.d/hooks
```

Create hooks for both the `linux` and `grub` packages

```shell
/etc/pacman.d/hooks/99-secureboot-linux.hook
```
```shell
[Trigger]
Operation = Install
Operation = Upgrade
Type = Package
Target = linux

[Action]
Description = Signing Kernel for SecureBoot
When = PostTransaction
Exec = /usr/bin/find /boot/ -maxdepth 1 -name 'vmlinuz-*' -exec /usr/bin/sh -c 'if ! /usr/bin/sbverify --list {} 2>/dev/null | /usr/bin/grep -q "signature certificates"; then /usr/bin/sbsign --key /root/db.key --cert /root/db.crt --output {} {}; fi' \ ;
Depends = sbsigntools
Depends = findutils
Depends = grep
```

```shell
/etc/pacman.d/hooks/98-secureboot-grub.hook
```
```shell
[Trigger]
Operation = Install
Operation = Upgrade
Type = Package
Target = grub

[Action]
Description = Signing GRUB for SecureBoot
When = PostTransaction
Exec = /usr/bin/find /efi/ -name 'grubx64*' -exec /usr/bin/sh -c 'if ! /usr/bin/sbverify --list {} 2>/dev/null | /usr/bin/grep -q "signature certificates"; then /usr/bin/sbsign --key /root/db.key --cert /root/db.crt --output {} {}; fi' \ ;
Depends = sbsigntools
Depends = findutils
Depends = grep
```

#### Enroll keys in firmware
##### Copy all `*.cer`, `*.esl`, `*.auth` to the EFI system partition
```shell
cp /root/*.cer /root/*.esl /root/*.auth /efi/
```

##### Boot into UEFI firmware setup utility (frequently but incorrectly referred to as "BIOS")
```shell
systemctl reboot --firmware
```

Firmwares have various different interfaces, see [Replacing Keys Using Your Firmware's Setup Utility](http://www.rodsbooks.com/efi-bootloaders/controlling-sb.html#setuputil) if the following instructions are unclear or unsuccessful.

##### Set OS Type to `Windows UEFI mode`
Find the Secure Boot options and set OS Type to `Windows UEFI mode` (yes, even if we're not on Windows.) This may be necessary for Secure Boot to function.

##### Clear preloaded Secure Boot keys

Using Key Management, clear all preloaded Secure Boot keys (Microsoft and OEM).

By clearing all Secure Boot keys, you will enter into Setup Mode (so you can enroll your own Secure Boot keys).

##### Set or append the new keys
The keys must be set in the following order:

```shell
db => KEK => PK
```

This is due to some systems exiting setup mode as soon as a `PK` is entered.

Do not load the factory defaults, instead navigate the available filesystems in search of the files previously copied to the EFI System partition.

Choose any of the formats. The firmware should prompt you to enter the type (*Note:* type names may differ slightly.)
```
*.cer is a Public Key Certificate
*.esl is a UEFI Secure Variable
*.auth is an Authenticated Variable
```

Certain firmware (such as my own) require you use the *.auth files. Try various ones until they work.

##### Set UEFI supervisor (administrator) password
You must also set your UEFI firmware supervisor (administrator) password in the Security settings, so nobody can simply boot into UEFI setup utility and turn off Secure Boot.

You should never use the same UEFI firmware supervisor password as your encryption password, because on some old laptops, the supervisor password could be recovered as plain text from the EEPROM chip.

##### Exit and save changes
Once you've loaded all three keys and set your supervisor password, hit F10 to exit and save your changes.

If everything was done properly, your boot loader should appear on reboot.

#### Check if Secure Boot was enabled
```shell
od -An -t u1 /sys/firmware/efi/efivars/SecureBoot-XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX
```
The characters denoted by XXXX differ from machine to machine. To help with this, you can use tab completion or list the EFI variables.

If Secure Boot is enabled, this command returns 1 as the final integer in a list of five, for example:

```
6  0  0  0  1
```

If Secure Boot was enabled and your UEFI supervisor password set, you may now consider yourself protected against Evil Maid attacks.

### Docker on BTRFS Storage Driver

[Docker on Arch](https://wiki.archlinux.org/title/docker).

BTRFS is ideal for docker snapshots. Follow settings by this [original article](https://docs.docker.com/storage/storagedriver/btrfs-driver/).

