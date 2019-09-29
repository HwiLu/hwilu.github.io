---
layout: post
title: RedHat/Centos解压缩 rar 文件
categories: [linux]
description: RedHat/Centos解压缩 rar 文件
keywords: linux
---

# 下载对应的工具



我们使用 rar for linux x64 工具进行操作。[官网](https://www.rarlab.com/download.htm)

选择下载 rarlinux-x64-5.6.0.tar.gz ，或者其他更加新的版本。

解压安装

```shell
# tar -zvxf rarlinux-x64-5.6.0.tar.gz 
# cd rarlinux-x64-5.6.0/
# make
# make install
```

现在便可以使用 rar 和 unrar 命令了。

解压文件命令

```shell
# unrar x [file] 
```

# 使用时报错

1. 如果在使用时出现 ` unrar: error while loading shared libraries: libstdc++.so.6: cannot open shared object file: No such file or directory`

安装`libstdc++.i686`加以解决。

```
# yum install libstdc++.i686

```



2. 如果出现 `[/lib/ld-linux.so.2: bad ELF interpreter: No such file or directory](https://stackoverflow.com/questions/14030306/lib-ld-linux-so-2-bad-elf-interpreter-no-such-file-or-directory)`

`yum install glibc.i686` 加以解决