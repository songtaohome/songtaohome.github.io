---
layout: post
title: 解决Lost connection to MySQL server at reading initial communication packet
categories: mysql
description: 解决Lost connection to MySQL server at reading initial communication packet
keywords: mysql
---

当通过 TCP/IP 连接 MySQL 远程主机时，出现 `ERROR 2013(HY000): Lost connection to MySQL server at 'reading initial communication packet', system error: 111` 。

如果是在linux 命令行中直接打 mysql 命令，能够顺利连上 MySQL，执行

查询语句也正常。

但是如果是使用TCP方式的远程连接情况下，则必然出现 `Lost connection to MySQL server at 'reading initial communication packet', system error: 111`。

要是无论通过什么途径远程访问都出现错误可以认为是系统有防火墙之类的限

制，但现在这种奇怪的抽筋现象让人百思不得其解。最后找到的解决方法是在 

`my.cnf` 里面的 [mysqld] 段增加一个启动参数  

`skip-name-resolve`  

问题消失。但原因还是想不出所以然。 

**产生的原因是 my.cnf 中我设置了 skip-name-resolve，skip-name-resolve是禁用dns解析，所以在mysql的授权表中就不能使用主机名了，只能使用IP** 。




