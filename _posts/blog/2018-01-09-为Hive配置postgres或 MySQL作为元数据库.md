---
layout: post
title:为Hive配置PostgreSQL或MySQL数据库作为元数据库
categories: [Hive]
description: some word here
keywords: Hive
---

HIVE的元数据默认使用derby作为存储DB，derby作为轻量级的DB，在开发、测试过程中使用比较方便，但是在实际的生产环境中，还需要考虑易用性、容灾、稳定性以及各种监控、运维工具等，这些都是derby缺乏的。MySQL和PostgreSQL是两个比较常用的开源数据库系统，在生产环境中比较多的用来替换derby。

## PostgreSQL

### 安装postgresql

找一个合适的节点

 

```
# yum install postgresql postgresql-contrib
# su - postgres
# initdb 
```

- 启动：

 

```
    postgres -D /var/lib/pgsql/data
or
    pg_ctl -D /var/lib/pgsql/data -l logfile start
```

- 创建HIve库

 

```
psql
```

 

```
CREATE USER hive WITH PASSWORD 'hive';
create database hive;
grant all privileges ON DATABASE hive to hive;
```

### 设置Postgresql 数据库

### 修改pg_hba.conf文件

修改 /etc/postgresql/10/main/pg_hba.conf文件

 

```
# Database administrative login by Unix domain socket
#local all postgres peer
local all postgres trust

# TYPE DATABASE USER ADDRESS METHOD

# "local" is for Unix domain socket connections only
#local all all peer
local all all trust
# IPv4 local connections:
#host all all 127.0.0.1/32 md5
host all all 127.0.0.1/32 trust        #允许本地登陆
host all all 192.168.158.0/24 trusr    # 允许从 192.168.158. 网段的主机登陆；这里为了安全，可以只设置为Hive metastore主机ip即可。
# IPv6 local connections:
#host all all ::1/128 md5
host all all ::1/128 trust
# Allow replication connections from localhost, by a user with the
# replication privilege.
#local replication all peer
#local replication all peer
#local replication all peer
local replication all trust
host replication all 127.0.0.1/32 trust
host replication all ::1/128 trust
```

### 下载PG的JDBC驱动

https://jdbc.postgresql.org/download.html

需下对应java版本的jdbc驱动：

![img](file:///C:/Users/24189/Documents/My Knowledge/temp/54d3a734-bc55-47fe-a915-6f0f91f6c62d/128/index_files/c2e19d7d-1d65-4868-a9f5-206a6dbfbfa9.png)

放在ambari-server 所在节点。

如何JDBC版本使用错误，metastore日志将会报如下错误：

 

```
2019-12-02T20:56:01,857 ERROR [main]: metastore.HiveMetaStore (HiveMetaStore.java:main(9316)) - Metastore Thrift Server threw an exception...
org.apache.hadoop.hive.metastore.api.MetaException: Error creating transactional connection factory
        at org.apache.hadoop.hive.metastore.RetryingHMSHandler.<init>(RetryingHMSHandler.java:84) ~[hive-exec-3.1.0.3.0.1.0-187.jar:3.1.0.3.0.1.0-187]
        at org.apache.hadoop.hive.metastore.RetryingHMSHandler.getProxy(RetryingHMSHandler.java:93) ~[hive-exec-3.1.0.3.0.1.0-187.jar:3.1.0.3.0.1.0-187]
        at org.apache.hadoop.hive.metastore.HiveMetaStore.newRetryingHMSHandler(HiveMetaStore.java:9129) ~[hive-exec-3.1.0.3.0.1.0-187.jar:3.1.0.3.0.1.0-187]
        at org.apache.hadoop.hive.metastore.HiveMetaStore.newRetryingHMSHandler(HiveMetaStore.java:9124) ~[hive-exec-3.1.0.3.0.1.0-187.jar:3.1.0.3.0.1.0-187]
        at org.apache.hadoop.hive.metastore.HiveMetaStore.startMetaStore(HiveMetaStore.java:9394) ~[hive-exec-3.1.0.3.0.1.0-187.jar:3.1.0.3.0.1.0-187]
        at org.apache.hadoop.hive.metastore.HiveMetaStore.main(HiveMetaStore.java:9311) [hive-exec-3.1.0.3.0.1.0-187.jar:3.1.0.3.0.1.0-187]
        at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method) ~[?:1.8.0_191]
        at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62) ~[?:1.8.0_191]
        at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43) ~[?:1.8.0_191]
        at java.lang.reflect.Method.invoke(Method.java:498) ~[?:1.8.0_191]
        at org.apache.hadoop.util.RunJar.run(RunJar.java:318) [hadoop-common-3.1.1.3.0.1.0-187.jar:?]
        at org.apache.hadoop.util.RunJar.main(RunJar.java:232) [hadoop-common-3.1.1.3.0.1.0-187.jar:?]
Caused by: org.apache.hadoop.hive.metastore.api.MetaException: Error creating transactional connection factory
        at org.apache.hadoop.hive.metastore.RetryingHMSHandler.invokeInternal(RetryingHMSHandler.java:208) ~[hive-exec-3.1.0.3.0.1.0-187.jar:3.1.0.3.0.1.0-187]
        at org.apache.hadoop.hive.metastore.RetryingHMSHandler.invoke(RetryingHMSHandler.java:108) ~[hive-exec-3.1.0.3.0.1.0-187.jar:3.1.0.3.0.1.0-187]
        at org.apache.hadoop.hive.metastore.RetryingHMSHandler.<init>(RetryingHMSHandler.java:80) ~[hive-exec-3.1.0.3.0.1.0-187.jar:3.1.0.3.0.1.0-187]
        ... 11 more
Caused by: javax.jdo.JDOFatalInternalException: Error creating transactional connection factory
        at org.datanucleus.api.jdo.NucleusJDOHelper.getJDOExceptionForNucleusException(NucleusJDOHelper.java:671) ~[datanucleus-api-jdo-4.2.4.jar:?]
        at org.datanucleus.api.jdo.JDOPersistenceManagerFactory.freezeConfiguration(JDOPersistenceManagerFactory.java:830) ~[datanucleus-api-jdo-4.2.4.jar:?]
        at org.datanucleus.api.jdo.JDOPersistenceManagerFactory.createPersistenceManagerFactory(JDOPersistenceManagerFactory.java:334) ~[datanucleus-api-jdo-4.2.4.jar:?]
        at org.datanucleus.api.jdo.JDOPersistenceManagerFactory.getPersistenceManagerFactory(JDOPersistenceManagerFactory.java:213) ~[datanucleus-api-jdo-4.2.4.jar:?]
        at sun.reflect.GeneratedMethodAccessor4.invoke(Unknown Source) ~[?:?]
        at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43) ~[?:1.8.0_191]
        at java.lang.reflect.Method.invoke(Method.java:498) ~[?:1.8.0_191]
        at javax.jdo.JDOHelper$16.run(JDOHelper.java:1965) ~[hive-exec-3.1.0.3.0.1.0-187.jar:3.1.0.3.0.1.0-187]
        at java.security.AccessController.doPrivileged(Native Method) ~[?:1.8.0_191]
.......
.......

Caused by: com.zaxxer.hikari.pool.HikariPool$PoolInitializationException: Failed to initialize pool: Method org.postgresql.jdbc4.Jdbc4Connection.isValid(int) is not yet implemented.
        at com.zaxxer.hikari.pool.HikariPool.throwPoolInitializationException(HikariPool.java:544)
        at com.zaxxer.hikari.pool.HikariPool.checkFailFast(HikariPool.java:529)
        at com.zaxxer.hikari.pool.HikariPool.<init>(HikariPool.java:112)
        at com.zaxxer.hikari.HikariDataSource.<init>(HikariDataSource.java:72)
        at org.datanucleus.store.rdbms.connectionpool.HikariCPConnectionPoolFactory.createConnectionPool(HikariCPConnectionPoolFactory.java:176)
        at org.datanucleus.store.rdbms.ConnectionFactoryImpl.generateDataSources(ConnectionFactoryImpl.java:213)
        ... 61 more
Caused by: org.postgresql.util.PSQLException: Method org.postgresql.jdbc4.Jdbc4Connection.isValid(int) is not yet implemented.
        at org.postgresql.Driver.notImplemented(Driver.java:753)
        at org.postgresql.jdbc4.AbstractJdbc4Connection.isValid(AbstractJdbc4Connection.java:109)
        at org.postgresql.jdbc4.Jdbc4Connection.isValid(Jdbc4Connection.java:21)
```

后面在stackoverflow上才找到答案https://stackoverflow.com/questions/25594279/method-org-postgresql-jdbc4-jdbc4connection-isvalidint-is-not-yet-implemented

在ambari服务那里点击添加新服务，点击添加Hive组件，ambari会提示如果需要使用postgresql数据库，需使用以下命令来加载驱动：

 

```
ambari-server setup --jdbc-db=postgres --jdbc-driver=/path/to/postgresql-9.0-801.jdbc4.jar
```

运行：

 

```
ambari-server setup --jdbc-db=postgres --jdbc-driver=/usr/local/jdk/jdk1.8.0_191/lib/postgresql-9.0-801.jdbc4.jar
```

运行之后会发现，该集群已经被复制到Hive metastore的安装目录下的lib目录下了。

### 测试连接

在ambari上填写postgresql的hive用户及库信息。点击测试连接，ok即可以下一步安装。

### 启动Hive

有看到说需要初始化数据库。

 

```
先运行schematool进行初始化：

schematool -dbType postgres -initSchema
然后执行$ hive 启动hive。
```



但是，发现该步骤不进行也可以。

## MySQL

如果需要使用MySQL作为meta store后台数据，按照以下步骤进行配置。

### 安装MySQL

同样，选择一个合适的节点。

安装教程参考安装mariadb那篇笔记。

登陆之后

 

```
> CREATE USER 'hive' IDENTIFIED BY '5tgb%TGB';
> CREATE DATABASE hive;
> GRANT ALL PRIVILEGES ON hive.* TO hive@’%’ IDENTIFIED BY ’hive’;
> GRANT ALL PRIVILEGES ON hive.* TO hive@’localhost’ IDENTIFIED BY ’hive’;
> flush privileges;
```

安装驱动

下载下面这个文件。

`mysql-connector-java-5.1.43.jar`

放于/usr/share/java 目录之下

并将该驱动复制到hive安装目录的lib目录下。

```
ln -s /usr/share/java/mysql-connector-java-5.1.45.jar /PATH/TO/HIVE/lib
```

### 安装Hive

在ambari上持续点击下一步，配置数据库。

然后运行：

 

```
ambari-server setup --jdbc-db=mysql --jdbc-driver=/usr/share/java/mysql-connector-java.jar
```

点击 test connect 看是否可以连接成功。

如果可以。

点击下一步，进行hive的安装。

有看到说需要初始化数据库。

 

```
PATH/TO/HIVE/bin/schematool -dbType mysql -initSchema
```

但是，发现该步骤不进行也可以。



Reference:https://juejin.im/post/5d023446e51d4555fc1acc8d