---
title: arch-install-archinstall-luks-btrfs
tags:
  - Notebooks/Geek_Room
created: 2024-10-14 08:49
modified: 2023-11-23T18:06:53.440Z
moc: "[[_geek-room]]"
---
# arch-install-archinstall-luks-btrfs

Fast installation with `archinstall` script. Full record available in youtube raven2cz channel in a geek-room playlist.

## Example of configuration

```json
{
    "HSM": null,
    "__separator__": null,
    "additional-repositories": [],
    "archinstall-language": "Czech",
    "audio": "pipewire",
    "bootloader": "grub-install",
    "config_version": "2.5.1",
    "debug": false,
    "desktop-environment": "awesome",
    "gfx_driver": "VMware / VirtualBox (open-source)",
    "harddrives": [
        "/dev/sda"
    ],
    "hostname": "r5arch",
    "kernels": [
        "linux"
    ],
    "keyboard-layout": "us",
    "mirror-region": {
        "Czechia": {
            "http://ftp.fi.muni.cz/pub/linux/arch/$repo/os/$arch": true,
            "http://ftp.linux.cz/pub/linux/arch/$repo/os/$arch": true,
            "http://ftp.sh.cvut.cz/arch/$repo/os/$arch": true,
            "http://gluttony.sin.cvut.cz/arch/$repo/os/$arch": true,
            "http://mirror.dkm.cz/archlinux/$repo/os/$arch": true,
            "http://mirror.vpsfree.cz/archlinux/$repo/os/$arch": true,
            "http://mirrors.nic.cz/archlinux/$repo/os/$arch": true,
            "https://europe.mirror.pkgbuild.com/$repo/os/$arch": true,
            "https://ftp.sh.cvut.cz/arch/$repo/os/$arch": true,
            "https://gluttony.sin.cvut.cz/arch/$repo/os/$arch": true,
            "https://mirror.dkm.cz/archlinux/$repo/os/$arch": true,
            "https://mirrors.nic.cz/archlinux/$repo/os/$arch": true
        }
    },
    "nic": {
        "dhcp": true,
        "dns": null,
        "gateway": null,
        "iface": null,
        "ip": null,
        "type": "nm"
    },
    "no_pkg_lookups": false,
    "ntp": true,
    "offline": false,
    "packages": [
        "polkit"
    ],
    "parallel downloads": 0,
    "profile": {
        "path": "/usr/lib/python3.10/site-packages/archinstall/profiles/desktop.py"
    },
    "save_config": null,
    "script": "guided",
    "silent": false,
    "swap": false,
    "sys-encoding": "UTF-8",
    "sys-language": "en_US",
    "timezone": "UTC",
    "version": "2.5.1"
}
{
    "/dev/sda": {
        "partitions": [
            {
                "boot": true,
                "encrypted": false,
                "filesystem": {
                    "format": "fat32"
                },
                "mountpoint": "/boot",
                "size": "512MiB",
                "start": "1MiB",
                "type": "primary",
                "wipe": true
            },
            {
                "btrfs": {
                    "subvolumes": [
                        {
                            "compress": false,
                            "mountpoint": "/",
                            "name": "@",
                            "nodatacow": false
                        },
                        {
                            "compress": false,
                            "mountpoint": "/home",
                            "name": "@home",
                            "nodatacow": false
                        },
                        {
                            "compress": false,
                            "mountpoint": "/var/log",
                            "name": "@log",
                            "nodatacow": false
                        },
                        {
                            "compress": false,
                            "mountpoint": "/var/cache/pacman/pkg",
                            "name": "@pkg",
                            "nodatacow": false
                        },
                        {
                            "compress": false,
                            "mountpoint": "/.snapshots",
                            "name": "@.snapshots",
                            "nodatacow": false
                        }
                    ]
                },
                "encrypted": true,
                "filesystem": {
                    "format": "btrfs",
                    "mount_options": [
                        "compress=zstd"
                    ]
                },
                "generate-encryption-key-file": true,
                "mountpoint": null,
                "size": "100%",
                "start": "513MiB",
                "type": "primary",
                "wipe": true
            }
        ],
        "wipe": true
    }
}
```
