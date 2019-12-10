---
layout: post
title: 如何将Hive元数据库从PostgreSQL迁移至MySQL
categories: [hive]
description: some word here
keywords: hive
---

在准备HDPCA 考试时，听闻Hive底层数据库使用的是pg，于是在测试环境配置了pg数据库作为其底层数据库，之后考虑到需要统一ambari、ranger数据库到MySQL，便想要测试一下如果Hive内本身存在业务数据时，需要如何将其元数据从pg迁移至MySQL。

首先，我们准备一下测试数据：

```
create table testtable (id int, name string,age int, tel string) ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t' STORED AS TEXTFILE;
```

```
# cat testdata.txt 
1 mayun 25 13188888888888
2 mahuateng 30 13888888888888
3 majiaying 34 899314121
```

加载数据

```
hdfs dfs -put testdata.txt /tmp/load data inpath 'testdata.txt' into table testtable;
```

查看数据

```
>select * from testtable;
+---------------+-----------------+----------------+-----------------+
| testtable.id  | testtable.name  | testtable.age  |  testtable.tel  |
+---------------+-----------------+----------------+-----------------+
| NULL          | NULL            | NULL           | NULL            |
| NULL          | NULL            | NULL           | NULL            |
| NULL          | NULL            | NULL           | NULL            |
| NULL          | NULL            | NULL           | NULL            |
| 1             |  mayun          | 25             | 13188888888888  |
| 2             | mahuateng       | 30             | 13888888888888  |
| 3             | majiaying       | 34             | 899314121       |
+---------------+-----------------+----------------+-----------------+
```

数据准备完毕之后，我们开始迁移操作。因为业务数据较少，所以整个过程都比较快，业务数据较大的情况尚未测试。

1、停止Hive服务，以保证在迁移过程中数据不会发生变化；
2、备份postgres的hive元数据库，包括数据和视图；

```
su - postgrespg_dump hive > hive.bak
```

3、在MySQL内为Hive创建元数据库；

```
> CREATE USER 'hive' IDENTIFIED BY '2wsx@WSX';> CREATE DATABASE hive;> GRANT ALL PRIVILEGES ON hive.* TO hive@’%’ IDENTIFIED BY ’hive’;> GRANT ALL PRIVILEGES ON hive.* TO hive@’localhost’ IDENTIFIED BY ’hive’;> flush privileges;
```

4、在Ambari或者CM上配置Hive连接到MySQL，并启动Hive，使其创建元数据表和视图；
切换完之后，Hive也是可以启动的。

![img](/images/posts/hive/hive-metastore-ri-正常启动.png)

我们可以登入到MySQL内可以看到元数据表格已经被创建。
但是切完之后，不把元数据导入是无法查到原来有的业务数据的，如下图所示：

![img](/images/posts/hive/result-null.png)

5、使用以下命令导出pg内Hive元数据库;

```
pg_dump -U <username> --column-inserts --format p -h <postgre-host> -p <port-number> -f hive-pg.sql -a <database> # -a表示只导出数据文件# -a表示只导出数据文件
```

6、使用pg转MySQL的转换工具做从pg到MySQL的SQL转换，推荐使用`https://github.com/ChrisLundquist/pg2mysql` 工具；大数据量不太建议使用；
7、安装php-cli工具，如果使用上述工具，使用以下命令进行SQL转换；

```
php pg2mysql-1.9/pg2mysql_cli.php hive-pg.sql hive-mysql.sql InnoDB
```

这里需要注意的是，pg2mysql如果php版本大于5.3，将会报以下错误：

```
php pg2mysql-1.9/pg2mysql_cli.php hive-pg.sql hive-mysql.sqlPHP Fatal error: Call-time pass-by-reference has been removed in /var/lib/pgsql/pg2mysql-1.9/pg2mysql.inc.php on line 133
```

在linux7 上，yum安装的php是5.4版本，于是，我们可以将其在centos6上执行转换成功：

```
# php pg2mysql-1.9/pg2mysql_cli.php hive-pg.sql hive-mysql.sql InnoDBFilesize: 590.7KPHP Warning:  date(): It is not safe to rely on the system's timezone settings. You are *required* to use the date.timezone setting or the date_default_timezone_set() function. In case you used any of those methods and you are still getting this warning, you most likely misspelled the timezone identifier. We selected 'America/Los_Angeles' for 'PST/-8.0/no DST' instead in /root/pg2mysql-1.9/pg2mysql.inc.php on line 141Completed!      2870 lines        2399 sql chunksNotes: - No its not perfect - Yes it discards ALL stored procedures - Yes it discards ALL queries except for CREATE TABLE and INSERT INTO  - Yes you can email us suggestsions: info[AT]lightbox.org    - In emails, please include the Postgres code, and the expected MySQL code - If you're having problems creating your postgres dump, make sure you use "--format p --inserts" - Default output engine if not specified is MyISAM.
```

当然，也可以选择其他工具进行转换。

8、需要对hive-mysql.sql做相应的修改以使其适配MySQL；

```
SET FOREIGN_KEY_CHECKS=0; # 放至开头SET FOREIGN_KEY_CHECKS=1; # 放至结尾
```

使其在数据导入时不会检查外键；
其中有三个表格在MySQL内为大写，需要将以下SQL:

```
INSERT INTO next_compaction_queue_id (ncq_next) VALUES (1);INSERT INTO next_lock_id (nl_next) VALUES (1);INSERT INTO next_txn_id (ntxn_next) VALUES (1);
```

修改为：

```
INSERT INTO next_compaction_queue_id (ncq_next) VALUES (1);INSERT INTO next_lock_id (nl_next) VALUES (1);INSERT INTO next_txn_id (ntxn_next) VALUES (1);
```

9、将hive-mysql.sql 导入到所创建的Hive元数据库;

```
USE hive;SOURCE /path/to/hive-mysql.sql;
```

10、现在可以重新启动Hive并测试元数据是否迁移成功，业务数据是否丢失
切过去之后。发现数据可以查询出来了,并且与此前一致;

![img](/images/posts/hive/result-right.png)

由此可见，本次迁移成功，切换顺利。