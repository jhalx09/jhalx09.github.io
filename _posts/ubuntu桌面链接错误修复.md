---
title: ubuntu 桌面文件夹意外删除[resolved]
tags:
  - ubuntu
  - desktop
  - config
  - resolved
categories: linux
---

## 背景
因意外操作导致desktop文件夹被删除，ls命令查看/home/[username]/desktop显示链接失效；与此同时，由于desktop文件夹的缺失ubuntu系统自动将/home/[username]的内容自动显示在桌面上。

## 解决方案
```shell
cd ~/.config
vim user-dirs.dirs
```
根据文件内容，大致修改为以下内容，根据实际情况更改即可。
```text
# This file is written by xdg-user-dirs-update
# If you want to change or add directories, just edit the line you're
# interested in. All local changes will be retained on the next run.
# Format is XDG_xxx_DIR="$HOME/yyy", where yyy is a shell-escaped
# homedir-relative path, or XDG_xxx_DIR="/yyy", where /yyy is an
# absolute path. No other format is supported.
# 
XDG_DESKTOP_DIR="$HOME/Desktop"
XDG_DOWNLOAD_DIR="$HOME/Downloads"
XDG_TEMPLATES_DIR="$HOME/Templates"
XDG_PUBLICSHARE_DIR="$HOME/Public"
XDG_DOCUMENTS_DIR="$HOME/Documents"
XDG_MUSIC_DIR="$HOME/Music"
XDG_PICTURES_DIR="$HOME/Pictures"
XDG_VIDEOS_DIR="$HOME/Videos"
```
