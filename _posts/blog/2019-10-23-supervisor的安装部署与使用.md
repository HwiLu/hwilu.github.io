---
layout: post
title: supervisor 的安装部署与简单使用
categories: [supervisor]
description: supervisor入门第一篇
keywords: supervisor
---


supervisor 的安装部署与使用。

# 安装

如果可以连接外网，直接使用 `pip install supervisor` 安装即可。
内网安装需下载 tar 包本机编译安装。
安装包下载地址：[https://pypi.python.org/](https://pypi.python.org/) ，搜索 supervisor 即可。
下载之后，解压编译。

```
tar zxf supervisor-3.4.0.tar.gz 
cd supervisor-3.4.0
python setup.py install
```
创建一个 supervisor 配置文件存放路径。
```
mkdir -m 755 /etc/supervisor/
```

生成配置文件。

```
echo_supervisord_conf > /etc/supervisor/supervisord.conf
```
启动。

```
supervisord -c /etc/supervisor/supervisord.conf
```
查看进程。
```
 ps -ef |grep supervisor
root     22222     1  0 15:26 ?        00:00:02 /bin/python /bin/supervisord -c /etc/supervisor/supervisord.conf
```
可以先去掉它自带的 http 的注释来看一下它自带的 ui 界面，去掉配置文件中的以下行的注释：

```
[inet_http_server]         ; inet (TCP) server disabled by default
port=ip:9001        ; ip_address:port specifier, *:port for all iface
username=user              ; default is no username (open server)
password=123               ; default is no password (open server)
```
ip 改为自己的 ip。
在浏览器访问 http://ip:9001 即可。

配置 supervisord 进程开机自启动。

```
将安装包中的supervisord文件拷到/etc/init.d/中
cp supervisord /etc/init.d/
chmod +x /etc/init.d/supervisord
加入开机启动
chkconfig --add supervisord
查看开机启动配置情况
chkconfig --list |grep supervisord
```
# 添加监控

添加一个监控试试其功能，小试牛刀一下下。
比如，我需要将 kafkaOffset 的监控程序加入 supervisor 来管理。
在 `/etc/supervisor/supervisord.conf` 中添加如下配置：

```
[program:kafkaMonitor]
command=/bin/bash /root/kafkaOffsetMonitor.sh
user=root
startretries=1
autostart=true
killasgroup=true
stopasgroup=true
stdout_logfile=/var/log/kakfaMonotorSupervisor.log
stderr_logfile=/var/log/kakfaMonotorSupervisor.err
```

`killasgroup=true`  

`stopasgroup=true`

两个配置表示，在 stop 管理的进程时候是否杀掉其子进程。
经测试证明，如果不杀掉，其子进程将会变成孤儿进程。


**注：**  command 后面的命令和文件需要使用绝对路径。 后面使用又发现 `/root/kafkaOffsetMonitor.sh` 内的文件路径也需要使用绝对路径。

supervisor 只能管理在前台运行的程序，所以如果应用程序有后台运行的选项，需要关闭。

重载配置文件

```
supervisorctl reload
```

现在通过其 ui 界面看看其管理的进程情况

![supervisor-ui](/images/posts/supervisor/supervisor-ui.png)

可以看到已经有其监控了。

下面简要介绍 supervisor 相应的知识点。
# supervisor 知识点

## 简介

supervisor 是一个用 Python 写的进程管理工具，可以很方便的用来启动、重启、关闭进程。如服务器重启之后，对未添加自启动但是添加了 supervisor 管理的进程进行统一管理。

supervisor 分为4部分

**supervisord**

运行 Supervisor 时会启动一个进程 supervisord，它负责启动所管理的进程，并将所管理的进程作为自己的子进程来启动，而且可以在所管理的进程出现崩溃时自动重启。

**supervisorctl**

命令行管理工具

**Web Server**

supervisor 的 web 服务。

**XML-RPC Interface**

supervisor的 api 。

官方文档：[supervisord.org](http://supervisord.org/introduction.html)
## 一些管理命令

```
	开启supervisord服务
 supervisord -c /etc/supervisord.conf
　　更新新的配置到supervisord
 supervisorctl update
　　重新启动配置中的所有程序
 supervisorctl reload
　　启动某个进程 
 supervisorctl start program_name
 	停止所有程序进程
 supervisorctl stop all     
 
	管理所有属于名为 group 这个分组的进程
 sudo supervisorctl start/stop/restart/status group
 
    管理分组里指定的进程
 supervisorctl start/stop/restart/status group:name1    
```
也可以使用查看正在守候的进程 supervisorctl 进行supervisor shell后 `help` 查看其有哪些管理命令。

## 设置组

我们可以对一批进程设置组
```
;[group:thegroupname]
;programs=progname1,progname2  ; each refers to 'x' in [program:x] definitions
;priority=999                  ; the relative start priority (default 999)

; The [include] section can just contain the "files" setting.  This
; setting can list multiple files (separated by whitespace or
; newlines).  It can also contain wildcards.  The filenames are
; interpreted as relative to this file.  Included files *cannot*
; include files themselves.
```

**refer :**  [https://www.cnblogs.com/kevingrace/p/7525200.html](https://www.cnblogs.com/kevingrace/p/7525200.html) 

**更多配置直接看配置文件内容的注释吧。**
