---
layout: post
title: HBase命令行权限管理
categories: [HBase]
description: 本文介绍如何使用命令对HBase的namespace、Table进行权限管理
keywords: HBase
---

在没有使用ranger对HBase表进行管理时，如何使用 `hbase shell` 对HBase的命名空间和表进行权限管理呢？

# 启用HBase安全认证功能

```xml
<property>
     <name>hbase.security.authorization</name>
     <value>true</value>
</property>
```
配置之后需要重启HBase生效。
# 查看权限

help `user_permission` 展示用法：
```
Show all permissions for the particular user.
Syntax : user_permission <table>

Note: A namespace must always precede with '@' character.

For example:

    hbase> user_permission
    hbase> user_permission '@ns1' // 查看 namespace 权限
    hbase> user_permission '@.*' // 查看所有 namespace 权限
    hbase> user_permission '@^[a-c].*'
    hbase> user_permission 'table1' // 查看 table 权限
    hbase> user_permission 'namespace1:table1' // 查看 table 权限
    hbase> user_permission '.*'
    hbase> user_permission '^[A-C].*'

```

# 权限类型
```
Read (R)：读取数据权限
Write (W)：写数据权限
Execute (X)：执行协处理器的权限
Create (C)：创建/删除表等操作
Admin (A)：管理员权限，即最高权限
```
# 赋权

`help grant` 展示具体用法：
```
Grant users specific rights.
Syntax : grant <user>, <permissions> [, <@namespace> [, <table> [, <column family> [, <column qualifier>]]]

permissions is either zero or more letters from the set "RWXCA".
READ('R'), WRITE('W'), EXEC('X'), CREATE('C'), ADMIN('A')

Note: Groups and users are granted access in the same way, but groups are prefixed with an '@' 
      character. In the same way, tables and namespaces are specified, but namespaces are 
      prefixed with an '@' character.

For example:

    hbase> grant 'bobsmith', 'RWXCA' // 为用户赋权
    hbase> grant '@admins', 'RWXCA' // 为用户组赋权
    hbase> grant 'bobsmith', 'RWXCA', '@ns1'
    hbase> grant 'bobsmith', 'RW', 't1', 'f1', 'col1'
    hbase> grant 'bobsmith', 'RW', 'ns1:t1', 'f1', 'col1' // 粒度可以控制到 ColumnFamily 和 Cell 级别
```

# 回收权限

`help revoke` 展示具体用法：
```
Revoke a user's access rights.
Syntax : revoke <user> [, <@namespace> [, <table> [, <column family> [, <column qualifier>]]]]

Note: Groups and users access are revoked in the same way, but groups are prefixed with an '@' 
      character. In the same way, tables and namespaces are specified, but namespaces are 
      prefixed with an '@' character.

For example:

    hbase> revoke 'bobsmith' // 移除用户权限
    hbase> revoke '@admins' // 移除用户组权限
    hbase> revoke 'bobsmith', '@ns1'
    hbase> revoke 'bobsmith', 't1', 'f1', 'col1'
    hbase> revoke 'bobsmith', 'ns1:t1', 'f1', 'col1'
```