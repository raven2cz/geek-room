---
title: arch-install-awesomewm
tags: [Notebooks/Geek_Room]
favorited: true
created: 2022-12-02T08:58:10.438Z
modified: 2023-11-23T18:06:53.440Z
---

# arch-install-awesomewm

# AwesomeWM Raven2cz Ecosystem

```shell
# Virtualbox basics
sudo pacman -S virtualbox-guest-utils
sudo systemctl status vboxservice.service
VBoxClient-all  (activate all services virtualbox, not stateful!)
sudo systemctl restart vboxservice.service
sudo systemctl status vboxservice.service
# add shared disk to /mnt/local
sudo gpasswd -a box vboxsf 
poweroff
VBoxClient-all
# test shared clipboard, dynamic xorg resolution, shared folder /mnt/local...

# paru (aur helper)
mkdir -p ~/git/github && cd ~/git/github
sudo pacman -S --needed base-devel
git clone https://aur.archlinux.org/paru.git
cd paru
makepkg -si

# several basics packages ideas
sudo pacman -S xorg xorg-xinit xterm git polkit polkit-gnome alsa-utils pulseaudio pulseaudio-alsa pavucontrol pipewire pipewire-pulse

# optional basics packages ideas
sudo pacman -S ntfs-3g grub-customizer kitty alacritty wezterm firefox brave-bin qpwgraph helvum feh qimgv gnome-disk-utility xorg-server-xephyr

# fonts simple solution (Monospaced variant of San Francisco. Sourced directly from Apple, patched with the Nerd Fonts Patcher)
paru -S nerd-fonts-sf-mono ttf-opensans
# fonts (huge package solution)
paru -S ttf-google-fonts-git nerd-fonts-complete ttf-ms-win11 ttf-ms-fonts 

# DWM as backup WM
cd ~/git/github
git clone git://git.suckless.org/dwm
git clone git://git.suckless.org/st
git clone git://git.suckless.org/dmenu
cd dwm ## repeat for each dwm, st and dmenu
sudo make clean install ## install all 3 parts: dwm, st, dmenu

# raven2cz project dotfiles
cd ~/git/github
mc
cd dotfiles-raven2cz
cp .xinitrc ~
cp .Xresources ~
nvim ~/.xinitrc ## if dwm: comment setxkbmap -layout "us,cz" - DWM is using this keybindings!

# try DWM!
startx ~/.xinitrc dwm

# (optional) Grub2 Themes
cd ~/git/github
git clone https://github.com/vinceliuice/grub2-themes.git
sudo ./install.sh -t vimix -s 2k

# (optional) Display Manager (SDDM or Lightdm)
paru -S lightdm web-greeter
sudo nvim /etc/lightdm/lightdm.conf
# visit [Seat:*], uncomment 
greeter-session=web-greeter
sudo systemctl enable lightdm.service #restart
# copy appropriate xsessions files to /usr/share/xsessions folder
# I have my in ~/.root, check it

# terminals
sudo pacman -S zsh fish
chsh -s /usr/bin/zsh <user>
nvim ~/.zshrc
# continue with https://github.com/raven2cz/tux/tree/main/211112-shell-terminal

# WindowManager packages ideas
paru -S picom-git network-manager-applet volctl rofi lxappearance qt5ct kvantum pop-theme mugshot papirus-icon-theme

# my public-wallpapers
cd ~/git/github
git clone https://github.com/raven2cz/public-wallpapers
ln -s ~/git/github/public-wallpapers ~/Pictures

# (optional) my awesomewm project tutorial
#https://github.com/raven2cz/tux/tree/main/211207-awesome-ricing
# 1. backup your awesomewm configuration first
# 2. git clone repository to ~/.config/awesome
git clone git@github.com:raven2cz/awesomewm-config.git ~/.config/awesome
# 3. ensure prerequsities and dependencies
paru -S rofi # rofi similar app like d-menu
# my used wallpapers and event images
mkdir ~/Pictures/wallpapers && git clone git@github.com:raven2cz/public-wallpapers.git
# my global colorscheme switcher script
# make steps which are described in the raven2cz/global-colorscheme.git project, you need themes for your terminal mainly
mkdir ~/git/github && git clone git@github.com:raven2cz/global-colorscheme.git && cd ~/git/github/global-colorscheme && ./install.sh
# rofi project themes
mkdir ~/.config/rofi && git clone https://github.com/raven2cz/rofi-themes ~/.config/rofi

# window manager: awesome-git
paru -S awesome-git 

# next options: xfce, kde, gnome
# https://github.com/raven2cz/tux/tree/main/211012-xfce-instalace
# https://github.com/raven2cz/tux/tree/main/211026-xfce-ricing
# https://github.com/raven2cz/tux/tree/main/220327-kde
# https://github.com/raven2cz/tux/tree/main/220501-gnome
```

