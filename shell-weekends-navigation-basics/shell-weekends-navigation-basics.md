---
title: shell-weekends-navigation-basics
tags: [Notebooks/Geek_Room]
created: 2023-05-29T12:34:30.515Z
modified: 2023-11-23T18:06:53.439Z
---

# shell-weekends-navigation-basics

<!--ts-->
<!--te-->

## Basics

* <kbd>Ctrl + A</kbd> Move to row beginning.
* <kbd>Ctrl + E</kbd> Move to row ending.
* <kbd>Ctrl + U</kbd> Remove text from cursor to row beginning.
* <kbd>Ctrl + W</kbd> Remove last word.

```shell
# basics
pwd

# z extension
z Downloads
z git

# start applications
gio launch /usr/share/applications/*.desktop
gio launch ~/.local/share/applications/*.desktop

# aliases
... = cd ../..
..  = cd ..
```

## fzf search and history
* <kbd>ctrl + r</kbd> - for history
* <kbd>ctrl + alt + f</kbd> - for search fuzzy in local directory (fish)

## images
```shell
# wezterm and kitty has similar solution
wezterm imgcat /tmp/kitty-greeting
```
