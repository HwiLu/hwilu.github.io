---
layout: post
title: SaltStack 安装部署及简单使用
categories: [SaltStack]
description: SaltStack 安装部署及简单使用
keywords: SaltStack
---

SaltStack可根据不同业务进行配置集中化管理、分发文件、采集服务器数据、操作系统基础及软件包管理等，SaltStack是运维人员提高工作效率、规范业务配置与操作的利器。

SaltStack 采用 C/S模式，server端就是salt的master，client端就是minion，minion与master之间通过ZeroMQ消息队列通信

# saltstack部署

## 简介
介绍几个需要记住的特点：
1. c/s结构；
2. 主控端（Master）与被控端（Minion）基于证书认证，确保安全可靠的通信；
3. 支持 API 及自定义 Python 模块，轻松实现功能扩展；
4. c/s端采用ZMQ协议，ZeroMQ 是一款开源的消息队列软件，用于在 Minion 端与 Master 端建立系统通信桥梁；
5. 支持多个master和多级master;

## 架构

通信端口

4505：master通过4505端口，订阅本地发布的信息，比如执行的命令

4506：minion连接上4505端口，获取执行的命令后，再通过连接4506端口，返回给master

### 基础架构
一master多minion

### HA结构
#### multi-master
多master多minion，master管理的是相同的minion。

#### salt-syndic

多级架构，使用salt-syndic降低master负载。


## 安装

本文讲解如何离线安装。

先安装基础依赖包，在所规划作为master和minion的节点上均运行

```shell
yum install -y PyYAML python-crypto  python-jinja2  python-markupsafe   python-msgpack  python-cherrypy
```

其余安装包及依赖包下载地址：https://repo.saltstack.com/yum/redhat/7/x86_64/，根据不同操作系统选择不同的版本。

### salt-master

master节点需安装：
```
libsodium-1.0.16-1.el7.x86_64.rpm
libtomcrypt-1.17-23.el7.x86_64.rpm
libtommath-0.42.0-4.el7.x86_64 (1).rpm
openpgm-5.2.122-2.el7.x86_64.rpm
python2-futures-3.0.5-1.el7.noarch.rpm
python2-pycryptodomex-3.6.1-2.el7.x86_64.rpm
python-crypto-2.6.1-2.el7.x86_64.rpm
python-meld3-0.6.10-1.sdl7.x86_64.rpm
python-msgpack-0.4.6-1.el7.x86_64.rpm
python-psutil-2.2.1-1.el7.x86_64.rpm
python-tornado-4.2.1-1.el7.x86_64.rpm
python-zmq-15.3.0-3.el7.x86_64.rpm
salt-2019.2.0-1.el7.noarch.rpm
salt-api-2019.2.0-1.el7.noarch.rpm
salt-master-2019.2.0-1.el7.noarch.rpm
salt-minion-2019.2.0-1.el7.noarch.rpm
salt-repo-latest.el7.noarch.rpm
salt-ssh-2019.2.0-1.el7.noarch.rpm
zeromq-4.1.4-7.el7.x86_64.rpm
zeromq-devel-4.1.4-7.el7.x86_64.rpm
```
即将这些包上传至master服务器，如：/salt-master目录，使用`rpm -ivh *.rpm`进行安装。如果包依赖包缺少，需到上文链接下载相应的安装包进行安装，如无误，启动salt-master：

```shell
# systemctl start salt-master
# systemctl enable salt-master
```



### salt-minion

minion节点需安装
```
libsodium-1.0.16-1.el7.x86_64.rpm
libtomcrypt-1.17-23.el7.x86_64.rpm
libtommath-0.42.0-4.el7.x86_64 (1).rpm
openpgm-5.2.122-2.el7.x86_64.rpm
python2-futures-3.0.5-1.el7.noarch.rpm
python-crypto-2.6.1-2.el7.x86_64.rpm
python-msgpack-0.4.6-1.el7.x86_64.rpm
python-psutil-2.2.1-1.el7.x86_64.rpm
python-tornado-4.2.1-1.el7.x86_64.rpm
python-zmq-15.3.0-3.el7.x86_64.rpm
salt-2019.2.0-1.el7.noarch.rpm
salt-minion-2019.2.0-1.el7.noarch.rpm
zeromq-4.1.4-7.el7.x86_64.rpm
```
将这些包上传至master服务器，如：/salt-minion目录后，
```shell
# cd salt-minion
# rpm -ivh *.rpm
```
安装无报错之后可以将这个/salt-minion目录删掉。

先不要启动，给每个 minion 赋予一个 id，在所有 minion 上执行：

```shell
 # hostname > /etc/salt/minion_id
```
编辑： /etc/salt/minion，新增：
```shell
master:
      - 第一个master节点的主机名
      - 第二个master节点的主机名
```

启动salt-minion

```shell
# systemctl start salt-minion
# systemctl enable salt-minion
```

以后修改了minion配置之后需要重启minion。

现在可以在master节点上看到minion的信息了：

- 列出全部 minion
```shell
[root@ambari salt]#  salt-key -L
Accepted Keys:
Denied Keys:
Unaccepted Keys:
node02
Rejected Keys:
```
发现node02节点还是处于unaccepted

- 接受全部 minion 节点的连接
```shell
[root@ambari salt]#  salt-key -A
The following keys are going to be accepted:
Unaccepted Keys:
node02
Proceed? [n/Y] y
Key for minion node02 accepted.
```
-  再次查看全部 minion
```shell
[root@ambari salt]#  salt-key -L
Accepted Keys:
node02
Denied Keys:
Unaccepted Keys:
Rejected Keys:
```

### 简单使用

#### cmd模块

```shell
salt '*' cmd.run "free -g"
```
#### cp模块

```shell
salt '*' cp.get_file salt://path/to/file /minion/dest   
将主服务器file_roots指定位置下的文件复制到被控主机
```

And so on...


### 部署HA架构

#### multi-master

不同人员所使用不同的主机需要关机所有相同的主机，建议部署多个Master。

1.在挑选一个节点作为master，比如node03；安装salt-master。

2.将原来的master配置文件同步过来:

```shell
# scp /etc/salt/master root@node03:/etc/salt/
```
3.同步Master秘钥对

```shell
scp /etc/salt/pki/master/master.pem master.pub root@node03:/etc/salt/pki/master/ 
```


4.修改所有salt-minion配置文件/etc/salt/minion
修改为：

```shell
master:
 - ambari.site.com
 - node03.site.com
```

重启

```
systemctl restart salt-minion
```

启动第二个master。

验证第二个master：

```shell
[root@node03 master]# systemctl start salt-master
[root@node03 master]# salt-key -L
Accepted Keys:
Denied Keys:
Unaccepted Keys:
node02
Rejected Keys:
[root@node03 master]# salt-key -A
The following keys are going to be accepted:
Unaccepted Keys:
node02
Proceed? [n/Y] y
Key for minion node02 accepted.
[root@node03 master]# salt-key -L
Accepted Keys:
node02
Denied Keys:
Unaccepted Keys:
Rejected Keys:
[root@node03 master]# salt '*' cmd.run "free -g"
node02:
                  total        used        free      shared  buff/cache   available
    Mem:             15           0          14           0           0          14
    Swap:             7           0           7
  
```
总结：
1. Master配置文件要一样
2. Master file_root路径及状态文件要一样
3. Master 公钥和私钥要一样
4. 修改Minion配置中指定Master为列表形式
5. Master接受的minion_id key要保持同步，增删保持一致

#### salt-syndic

多级架构

结构：

![saltstack的HA架构](/images/posts/saltstack/saltstack-ha-structure.png)

工作流程：

- 充当 Minion，建立与高级别的 Master 的连接，订阅所有来自高级 Master 下发的任务。
- 接收高级 Master 下发的数据后，首先进行解密，解密完成后，将其扔到本地的 Master 接口上进行二次下发。
- Syndic 在进行二次下发之后，监听本地 Event 接口，获取所管理的 Minions 的返回。
- 将返回发送给高级 Master

##### 安装

上传salt-master、salt-syndic及其依赖包。`rpm -ivh *`
在 Syndic 节点
```
# vim /etc/salt/master
syndic_master: master节点ip
```
```
 vim /etc/salt/minion
 id: my_syndic #本机主机名
```
在master节点
```
	vim /etc/salt/master
	order_masters: True
```

在Sydnic节点启动两个服务

```shell
systemctl start salt-syndic
systemctl start salt-master
```

然后，修改Minion配置`/etc/salt/minion` 的master节点配置为Syndic节点。

```shell
master:
 - zk02.site.com
```

并重启salt-minion。

then, 在Syndic上接受Minion的key；

then，在Master节点接受Minion的key。

```shell
 salt-key -L
 salt-key -A
```



现在，可以在Syndic上看到所有Minon节点了，在Master上可以看到Syndic了，但是看不到Minion。装多个Syndic时，重新走一遍上述流程即可。不同的Syndic管理的主机可以重复。



## 网络策略怎么开

如果需要开网络策略

Master到Minion方向：开通4505

​	Master-->4505-->Syndic-->4505-->Minion

Minion到Master方向：开通4506

​	Minion-->4506-->Syndic-->4506-->Master

