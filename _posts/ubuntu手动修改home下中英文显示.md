---
title: ubuntu 手动修改home下中英文显示[resolved]
categories: linux
tags:
  - config
  - desktop
  - resolved
  - ubuntu
---

```shell
vim ~/.config/user-dirs.locale
```

打开后会发现文件内容即为当前系统的语言支持。  
改为目标语言即可：  
zh_CN对应中文  
zh_en对应英文
