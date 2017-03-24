---
layout: post
title: 服务端安装memcache
categories: Linux
description: 服务端安装memcache, 以及报错
keywords: Memcache
---

php开发环境已经搭建完毕, 然后再给php安装memcache的扩展. 主要用的是php自带的phpize命令.

# 服务端安装memcache

##获取源码包
```
[root@CentOS src]# wget http://memcached.org/files/memcached-1.4.33.tar.gz
```

##解压源码包
```
[root@CentOS src]# tar zxvf memcached-1.4.33.tar.gz 
```

##添加安装目录
```
[root@CentOS memcached-1.4.33]# mkdir /usr/local/memcache
```

##变以前配置
```
[root@CentOS memcached-1.4.33]# ./configure --prefix=/usr/local/memcache
```

报错:

```
...
...
checking for library containing gethostbyname... none required
checking for libevent directory... configure: error: libevent is required.  You can get it from http://www.monkey.org/~provos/libevent/

      If it's already installed, specify its path using --with-libevent=/dir/

[root@CentOS memcached-1.4.33]# 
```

错误原因:缺少依赖包` libevent`
下载`libevent`, 下载且安装
```
[root@CentOS memcached-1.4.33]# wget -P /usr/local/src/ https://github.com/libevent/libevent/releases/download/release-2.0.22-stable/libevent-2.0.22-stable.tar.gz
```

```
[root@CentOS memcache]# cd /usr/local/src
[root@CentOS src]# tar zxvf libevent-2.0.22-stable.tar.gz 
```

```
[root@CentOS src]# cd libevent-2.0.22-stable
```

```
[root@CentOS libevent-2.0.22-stable]# mkdir /usr/local/libevent
```

```
[root@CentOS libevent]# cd /usr/local/src
[root@CentOS src]# cd memcached-1.4.33
[root@CentOS memcached-1.4.33]# ./configure --prefix=/usr/local/memcache/ --with-libevent=/usr/local/libevent/
```

## 服务端的memcache安装完毕

```
[root@CentOS memcached-1.4.33]# wget -P /usr/local/src/ http://pecl.php.net/get/memcache-3.0.8.tgz
```

```
[root@CentOS memcached-1.4.33]# cd /usr/local/src
[root@CentOS src]# ls
libevent-2.0.22-stable  libevent-2.0.22-stable.tar.gz  lnmp.source  memcache-3.0.8.tgz  memcached-1.4.33  memcached-1.4.33.tar.gz
[root@CentOS src]# 
[root@CentOS src]# tar zxvf memcache-3.0.8.tgz
```



```
[root@CentOS memcache-3.0.8]# /usr/local/php5/bin/phpize 
Configuring for:
PHP Api Version:         20121113
Zend Module Api No:      20121212
Zend Extension Api No:   220121212
[root@CentOS memcache-3.0.8]# ./configure --with-php-config=/usr/local/php/bin/php-config --enable-memcache
```

![-w676](/images/posts/14798824446515.jpg)


##重启nginx, phpinfo没有memcache, 但是查看php模块有. 
![](/images/posts/14798825390938.jpg)

## 所以重启php-fpm试试, 结果就有了.

![-w737](/images/posts/14798826117911.jpg)




![-w414](/images/posts/14798832118487.jpg)

![-w475](/images/posts/14798832351511.jpg)



	





