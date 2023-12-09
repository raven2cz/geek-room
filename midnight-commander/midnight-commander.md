---
title: midnight-commander
tags: [Notebooks/Geek_Room]
created: 2023-05-24T12:18:30.780Z
modified: 2023-11-23T18:06:53.439Z
---

![midnight-commander-logo](../attachments/midnight-commander-logo.png)

# midnight-commander

<!--ts-->
<!--te-->

# Mastering Midnight Commander: A Comprehensive Guide for Novice Linux Users

**Midnight Commander** (MC) is a feature-rich file manager that uses a text-based interface (**TUI**). It's popular among Unix-like **system users**, especially those using Arch Linux, Debian, thanks to its versatile features and intuitive dual-pane navigation system. MC supports a variety of protocols including FTP and SFTP, incorporates a flexible keybinding mechanism, and much more. This article provides insights, tips, and tricks for mastering Midnight Commander, helping Linux users to significantly enhance their productivity.

## Installing Midnight Commander on Arch Linux

Arch Linux uses the pacman package manager. To install Midnight Commander, you can use the following command:

```shell
sudo pacman -S mc
```

To launch Midnight Commander, create the alias in your favorite shell. Some distributions and apps can cause significant slow MC start. It is a reason for this alias. 

```shell
alias mc="mc --nosubshell"
```

To launch Midnight Commander, simply type `mc` in your terminal which call your alias. 
You'll be greeted with a dual-pane interface and a command line prompt at the bottom.

## Basic Configuration: Improve Look and Feel and Application Control

* `Options > Panel Options > Lynx-like motion` (active it: [x])
* user: `Options > Appearance > modarin256-defbg`, root: `Options > Appearance > modarin256root-defbg` (transparent background, need some compositor active)

## Keybindings: Accelerate your Productivity

Midnight Commander's power comes to the fore with its extensive set of keybindings, making it a highly efficient tool for file management. Here are some of the most useful ones:

* <kbd>F1 to F10</kbd> These function keys are used for various operations such as help, menu, view, edit, copy, rename/move, mkdir, delete, pull-down menu, and quit respectively.
* <kbd>Insert</kbd> Select files.
* <kbd>Tab</kbd> Switch between the two panels.
* <kbd>Ctrl + r</kbd> Refresh listing.
* <kbd>Alt + i</kbd> Open actual directory on inactive panel.
* <kbd>Ctrl + u</kbd> Swap panels.
* <kbd>Alt + t</kbd> Cycle panel view mode.
* <kbd>+/-</kbd> Tag/untag items by pattern.
* <kbd>*</kbd> Invert tagging of items
* <kbd>Alt + s</kbd> Search­/jump to by pattern.
* <kbd>Ctrl + o</kbd> Show full MC command line screen.
* <kbd>Alt + p/n</kbd> MC command history inline.
* <kbd>Alt + h</kbd> MC command history.
* <kbd>Ctrl + space</kbd> Calculate selected item sizes (.. calculates all item sizes)
* <kbd>Alt + .</kbd> Toggle hidden files.
* <kbd>Alt + Enter</kbd> Copy the currently selected file name(s) to the command line.
* <kbd>Alt + a</kbd> Paste active­/in­active panel directory path
* <kbd>Ctrl + r</kbd> Refresh the active panel.
* <kbd>Shift + F6</kbd> Rename selected item.
* <kbd>Shift + F4</kbd> Open new empty file with an internal editor.
* <kbd>Ctrl + x, then c</kbd> Open the chmod dialog for the current file.
* <kbd>Ctrl + x, then o</kbd> Open the chown dialog for the current file.
* <kbd>Ctrl + x, then s</kbd> Open a symbolic link creation dialog.

Remember, you can view the full list of keybindings anytime by pressing F1 (the Help function key) while in Midnight Commander.

## Tips and Tricks for Advanced Users

### Customize the Visual Appearance

Midnight Commander allows for extensive visual customization. You can change the colors of the interface by selecting a different skin. This can be done by navigating to the Options menu, then Appearance.

### Make Use of the Internal Editor and Viewer

Midnight Commander comes with an internal file viewer (F3) and editor (F4). These are useful tools for quickly viewing or editing a file without leaving the MC environment.

### Use Virtual File Systems

One of the powerful features of MC is the support for Virtual File Systems (VFS). For example, you can navigate compressed archives or remote systems via Shell link, FTP, SFTP, or SMB just as if they were local directories.

### Bash Integration

Midnight Commander integrates well with bash. You can use bash commands directly in MC's command line at the bottom of the interface. If you press <kbd>Ctrl + o</kbd>, you can hide the panels to access a full bash shell.

### Directory Hotlist

MC has a directory hotlist feature (<kbd>Ctrl + \ </kbd>) that stores a list of your favorite directories, allowing you to quickly jump to a directory without typing its full path.

Midnight Commander is a powerful file manager that offers extensive features and customization options. By familiarizing yourself with the keybindings and other tips mentioned in this guide, you can significantly improve your file management efficiency on Arch Linux.

