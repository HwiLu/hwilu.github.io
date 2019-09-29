---
layout: post
title: Linux查询用户和用户组的对应关系
categories: [Linux, shell]
description: Linux users and its groups
keywords: Linux, shell
---

见以下脚本：
```
#!/bin/bash
echo "用户名 用户主组 用户附属组"
for user in `awk -F':' '{if($3+0>500) print $1}' /etc/passwd`
do
         #id $user | grep -o '([^)]*)'  |sed 's/(//g' | sed 's/)//g' | xargs | awk -F " "  '{for (i=2;i<=NF;i++)printf("%s ", $i);print ""}'  | column -t 
         id $user | grep -o '([^)]*)'  |sed 's/(//g' | sed 's/)//g' | xargs  | awk '{$2="";print $0}'
done
```
run
```
sh user-id.sh | column -t
```


**阅读正则表达式，需从里往外看。**

如：`id $user | grep -o '([^)]*)' `
首先理解`[^)]`的意思：负值字符集合。匹配未包含的任意字符；即匹配未包含`)`的任意字符。之后加个`*`即可匹配出`(` `)`之内的所有内容。

`xargs`进行行列转换。
