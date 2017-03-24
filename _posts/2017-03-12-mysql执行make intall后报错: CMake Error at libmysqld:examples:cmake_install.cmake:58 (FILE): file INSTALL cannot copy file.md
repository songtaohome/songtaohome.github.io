---
layout: post
title: mysql执行make intall后报错: CMake Error at libmysqld/examples/cmake_install.cmake:58 (FILE): file INSTALL cannot copy file
categories: MySQL
description: mysql执行make intall后报错: CMake Error at libmysqld/examples/cmake_install.cmake:58 (FILE): file INSTALL cannot copy file
keywords: Linux, MySQL
---

# mysql执行make intall后报错: CMake Error at libmysqld/examples/cmake_install.cmake:58 (FILE): file INSTALL cannot copy file

在编译安装mysql的时候, 会报错. 一直找不到原因, 原本以为是因为权限的问题, 但是我用的是root用户. 应该没问题的. 这里吐槽一下. baidu和google真的不是一个等级的. 我用百度搜索答案, 根本找不到, 用google, 直接就找到了我想要的答案. 浪费了我一天的时间, 因为编译一次就需要50分钟. 靠的. 

```
CMake Error at libmysqld/examples/cmake_install.cmake:58 (FILE):
  file INSTALL cannot copy file
  "/usr/local/src/mysql-5.6.35/libmysqld/examples/mysqltest_embedded" to
  "/usr/local/mysql/bin/mysqltest_embedded".
Call Stack (most recent call first):
  cmake_install.cmake:108 (INCLUDE)

make: *** [install] 错误 1
```

http://blog.itpub.net/27099995/viewspace-1994443/

这里给出了答案. 是因为磁盘空间不足导致无法拷贝文件的. 也是醉了. 我用我的PD查看了一下磁盘空间. 分配的是8G, 然后在虚拟机里查看一下剩余的空间. `df -hl`, 剩余不到2个G, 可能就是因为这个的原因. 那么我再试试. 扩大一下硬盘的容量. 如果还出现这个原因, 我就还得继续熬夜搞定这个问题了. 

我先在原来的8G磁盘空间下再次安装mysql, 在安装之前, 我先查了一下磁盘空间的大小. 

```
[root@CentOS7 ~]# df -hl
文件系统        容量  已用  可用 已用% 挂载点
/dev/sda2       6.0G  3.9G  2.2G   65% /
devtmpfs        486M     0  486M    0% /dev
tmpfs           495M     0  495M    0% /dev/shm
tmpfs           495M  6.6M  488M    2% /run
tmpfs           495M     0  495M    0% /sys/fs/cgroup
tmpfs            99M     0   99M    0% /run/user/0
```

然后我就开始编译安装mysql5.6, 果然还是存在这个错误. 

```
-- Installing: /usr/local/mysql/bin/mysqltest_embedded
-- Installing: /usr/local/mysql/bin/mysql_client_test_embedded
CMake Error at libmysqld/examples/cmake_install.cmake:74 (FILE):
  file INSTALL cannot copy file
  "/usr/local/src/mysql-5.6.16/libmysqld/examples/mysql_client_test_embedded"
  to "/usr/local/mysql/bin/mysql_client_test_embedded".
Call Stack (most recent call first):
  cmake_install.cmake:110 (INCLUDE)


make: *** [install] 错误 1
[root@CentOS7 mysql-5.6.16]#
```

然后我就再查看一下磁盘的大小

```
[root@CentOS7 /]# df -hl
文件系统        容量  已用  可用 已用% 挂载点
/dev/sda2       6.0G  6.0G   20K  100% /
devtmpfs        486M     0  486M    0% /dev
tmpfs           495M     0  495M    0% /dev/shm
tmpfs           495M  6.6M  488M    2% /run
tmpfs           495M     0  495M    0% /sys/fs/cgroup
tmpfs            99M     0   99M    0% /run/user/0
[root@CentOS7 /]#
```

看来就是磁盘没空间了. 所以导致无法拷贝文件到安装的目录. 哎. 郁闷.

测试了一下. 我重新用PD虚拟机装了centos7.3, 然后分配的内存为16G, 还是按照原来的方式安装的mysql5.6, 没有报错. 安装成功. 已经就是这个原因了. 





