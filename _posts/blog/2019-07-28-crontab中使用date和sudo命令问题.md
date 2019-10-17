---
layout: post
title: crontab中使用date和sudo命令问题
categories: [Linux]
description: 
keywords: Linux
---

## date

习惯上是这样的

```
`date +"%Y%m%d_%H:%M"` 
$(date +"%Y%m%d_%H:%M")
```

在`crontab`下不起作用，需采用如下形式

```
 `date +"\%Y\%m\%d_\%H:\%M"` 
 $(date +"\%Y\%m\%d_\%H:\%M")
```

因为在`crontab`中`%`被认为是特殊字符，需要使用`\`进行转义。

## sudo

直接在`crontab`里以`sudo`执行命令无效，会提示` sudo: sorry, you must have a tty to run sudo` .需要修改`/etc/sudoers`，执行visudo或者`vim /etc/sudoers` 将`"Defaults requiretty"`这一行注释掉。因为`sudo`默认需要`tty`终端，而`crontab`里的命令实际是以无tty形式执行的。注释掉`"Defaults requiretty"`即允许以无终端方式执行`sudo`。

但是，这里关于安全性方面有一点需要注意：

关于该配置项，说明如下`Disable "ssh hostname sudo ", because it will show the password in clear.You have to run "ssh -t hostname sudo ".`

该配置的作用是禁止执行`"ssh hostname sudo "`，因为这种方式会将sudo密码以明文显示，你可以运行`"ssh -t hostname sudo "`来替代。开启的情况下，`"ssh hostname sudo "`无法执行成功，关闭了之后，就没有这一层的检查了。