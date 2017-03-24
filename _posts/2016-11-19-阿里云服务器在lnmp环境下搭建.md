---
layout: post
title: 阿里云服务器在lnmp环境下搭建wordpress博客站点
categories: Linux
description: 阿里云服务器在lnmp环境下搭建wordpress博客站点
keywords: Linux, wordpress
---

# 阿里云服务器在lnmp环境下搭建wordpress博客站点
2016年11月19日 星期六
author : qiuyu

## 说明
环境: LNMP
服务器: 阿里云
wordpress版本: 4.6.1中文版

## wordpress下载
下载地址 : [WordPress4.5.3中文版](https://cn.wordpress.org/wordpress-4.5.3-zh_CN.zip)

下载好之后, 解压, 在官网下载的是zip压缩包. 

```
[root@iZ28cww0nf3Z src]# unzip wordpress-4.5.3-zh_CN.zip
```

然后把解压后的目录复制到我搭建好的站点下

```
[root@iZ28cww0nf3Z src]# cp -r wordpress /data/www/blog
```

ok, 这样就可以了. 只要环境没问题. 那么就可以开始设置WordPress了. 

![-w1279](/images/posts/14795567680103.jpg)

点击 `现在就开始`, 进入页面:

![-w1279](/images/posts/14795568305610.jpg)

数据库我还没有创建, 那就先去创建一个数据库吧. 

```
[root@iZ28cww0nf3Z blog]# mysql -u root -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 29
Server version: 5.5.51-log Source distribution

Copyright (c) 2000, 2016, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> create database blog;
Query OK, 1 row affected (0.00 sec)

mysql> 
```

创建完毕, 我创建的库名是`blog`, 所以, 图中的数据库名也需要填入`blog`, 其他的按照提示信息填入即可. 

![-w1279](/images/posts/14795570307302.jpg)

点击`提交`按钮

哎呀, 出现问题了. 看图
![-w1279](/images/posts/14795572647892.jpg)
提示说不能写入, 第一个想法就是没有写权限. 
呵呵. 之前我把`/data/www/blog`赋予过权限. 但是我为了方便, 把blog目录给删掉了. 然后在复制的过程中直接改名为blog, 所以还需要重新赋予权限和组. 

```
[root@iZ28cww0nf3Z www]# chown www.www blog -R
[root@iZ28cww0nf3Z www]# chmod 700 blog -R
```

ok, 那么, 我在网页中, 后台网页, 然后重新点击`提交`. 

![-w1279](/images/posts/14795574699678.jpg)

这回就正常了. 我的判断是对的. 

点击`进行安装`
![-w1279](/images/posts/14795575785795.jpg)

现在就是开始设置博客的内容了. 中文版的就这个好处, 都能看懂. 呵呵. 按照提示填写即可. 

点击`安装WordPress`

![-w1279](/images/posts/14795577299917.jpg)

点击`登录`
输入用户名和密码, OK. 

![-w1279](/images/posts/14795577847202.jpg)






