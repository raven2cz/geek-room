---
title: shell-weekends-navigation-basics
tags: [Notebooks/Geek Room]
created: 2023-05-29T12:34:30.515Z
modified: 2023-06-15T10:34:34.072Z
---

# shell-weekends-navigation-basics

<!--ts-->
* [shell-weekends-navigation-basics](#shell-weekends-navigation-basics)
   * [Basics](#basics)
   * [fzf search and history](#fzf-search-and-history)
   * [images](#images)

<!-- Added by: box, at: Thu Jun 15 12:44:47 PM CEST 2023 -->

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