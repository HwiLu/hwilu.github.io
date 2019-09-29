---
layout: post
title: 为什么bash脚本里if语句的方括号后面需要空格
categories: [Shell]
description: "Why should there be a space after '[' and before ']' in Bash?"
keywords: Shell
---



## 为什么bash脚本里if语句的方括号后面需要空格？为什么空格在bash脚本里这么重要？

以前写bash脚本的时候，经常被`[` `]`前后需不需要空格搞昏，直到在Stack Overflow上看到了这个问题[传送门](https://stackoverflow.com/questions/9581064/why-should-there-be-a-space-after-and-before-in-bash)，才明白是怎么回事。
比如：

```
if  [$CHOICE -eq 1];
```
上述脚本是会报错的。需要改成如下：
```
if  [ $CHOICE -eq 1 ];
```
优秀答案中提到，只要能够明白 `[` 其实是个linux的内建命令，那一切都会清晰明了得多。
我们可以通过 `help [` 查看 `[` 的用法：

```
 help [
[: [ arg... ]
    Evaluate conditional expression.
    
    This is a synonym for the "test" builtin, but the last argument must
    be a literal `]', to match the opening `['.
```
可以看出 `[` **命令** 与 `test` 命令等同。

**refer:[why-should-there-be-a-space-after-and-before-in-bash](https://stackoverflow.com/questions/9581064/why-should-there-be-a-space-after-and-before-in-bash)**
