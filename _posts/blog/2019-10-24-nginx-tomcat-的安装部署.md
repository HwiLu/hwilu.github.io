---
layout: post
title: Nginx 及 Apache Tomcat 的安装及部署
categories: [Nginx,Tomcat]
description: 仅介绍安装，本文暂不介绍原理。
keywords: Nginx,Tomcat
---

# 安装Nginx

下载源码包进行编译安装，下载地址：[nginx.org](http://nginx.org/en/download.html)。

下载之后，解压安装：
```
./configure
make && make install
```
如果发现缺少依赖包，极有可能是以下依赖包，使用以下命令解决：

```
yum install -y pcre pcre-devel
```
默认将被安装在 `/usr/local/nginx` 目录下。
启动Nginx

```
sbin/nginx 
```
然后，通过 http://ip 便可以看到 Nginx的欢迎界面。
如果出现访问错误，直接看 `error.log` 和 `access.log` 两个配置文件寻求解决。我出现过因为主机 umask 值为 077 导致欢迎界面打不开的，后面看了 `error.log` 发现是 `/usr/local/nginx` 和 `/usr/local/nginx/html`目录权限问题，需要将`/usr/local/nginx` 权限修改为 644，`/usr/local/nginx/html` 目录及目录下文件修改为 644 才能解决。

## 控制nginx
为了重新加载配置文件，你可以停止或重启nginx，或是发送信号给主进程master。

```
nginx -s signal
```
signal信号的值可以是以下几个：

- quit -优雅的关闭
- reload -重新加载配置文件
- reopen -重新打开日志文件
- stop -立即快速关闭

# 安装Tomcat

安装 Tomcat 之前需要安装 jdk 。
## 安装 jdk
下载tar包

[http://www.oracle.com/technetwork/java/javase/downloads/index.html](http://www.oracle.com/technetwork/java/javase/downloads/index.html)

解压到指定目录
```
tar -zvxf jdk...tar.gz -C /cm/jdk
mv /cm/jdk/jdk.1.8.xxx .
```
修改环境变量

打开/etc/profile文件 `vi /etc/profile` ，在最后添加
```
export JAVA_HOME=/cm/jdk
export JRE_HOME=$JAVA_HOME/jre 
export CLASSPATH=.:$JAVA_HOME/lib:$JRE_HOME/lib 
export PATH=$JAVA_HOME/bin:$PATH　
```

保存退出后，执行source /etc/profile是修改的环境变量生效。

验证
```
# java -version
java version "1.8.0_144"
Java(TM) SE Runtime Environment (build 1.8.0_144-b01)
Java HotSpot(TM) 64-Bit Server VM (build 25.144-b01, mixed mode)
```

## 安装 Tomcat 

登录 [Tomcat org](http://tomcat.apache.org) 下载 Tomcat 合适版本。

解压tomcat

```
tar -zxvf apache-tomcat-8.5.47.tar.gz
mv apache-tomcat-8.5.47 /usr/local/tomcat(修改名称)
```

tomcat 常用命令

```
/usr/local/tomcat/bin/startup.sh(启动命令)
/usr/local/tomcat/bin/shutdown.sh(关闭命令)
ps -ef|grep java(查看tomcat进程)
kill -9 进程号(杀死进程)
tail -f /usr/local/tomcat/logs/catalina.out(查看tomcat日志)
```

打开 http://ip:8080 查看 tomcat 欢迎界面。

![Tocamt-welcome](/images/posts/tomcat/tomcat-welcome.png)

安装成功。