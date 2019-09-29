---
layout: post
title: Mac环境安装MySQL
categories: [Mac]
description: mac book install MySQL
keywords: Mac, MySQL
---

下载安装包进行下载和配置实在是太复杂，我喜欢包管理工具。所幸，Mac 上具有类似于centos的包管理工具，叫做brew。

# 1、安装brew
在终端输入：
```vim
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```
# 2、安装mysql
```vim
brew  install mysql
```
# 3、启动
```
mysql.server start
```
上面指令将安装最新版本的mysql。
使用brew  安装的软件，均安装在/usr/local/Cellar 下。
# 4、配置mysql root 登陆密码
以上步骤走完之后，mysql可使用root免密登陆，如果需要为root设置密码，在终端使用以下指令：
  ```vim
mysql_secure_installation
  ```
按照提示一步一来即可，可提高mysql安全性。
# 5、安装指定版本的MySQL
直接使用brew  install mysql安装的是最新版本的mysql，如果需要安装指定版本的软件。
我们可以先使用brew search mysql 查看源内有哪些版本。然后根据版本好进行安装，如：
```vim
brew search mysql
==> Formulae
automysqlbackup              mysql++                      mysql-cluster                mysql-connector-c++          mysql-search-replace         mysql@5.5                    mysql@5.7
mysql ✔                      mysql-client                 mysql-connector-c            mysql-sandbox                mysql-utilities              mysql@5.6                    mysqltuner

==> Casks
homebrew/cask/mysql-connector-python    homebrew/cask/mysql-shell               homebrew/cask/mysql-utilities           homebrew/cask/navicat-for-mysql         homebrew/cask/sqlpro-for-mysql
```
如果需要安装5.7的mysql，可使用brew install mysql@5.7即可。