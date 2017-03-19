---
layout: post
title: 虚拟机下练习CentOS源码安装LNMP
categories: Linux
description: 虚拟机下练习CentOS源码安装LNMP
keywords: Linux, LNMP
---

之前看过一个评论, 说: 程序员, 谁不是安装几十上百次开发环境. 
so, 我又开始安装了. 

##环境准备

1. 操作系统安装：CentOS 6.3 32位最小化安装。(mini安装.)
2. 配置好IP、DNS、网关、主机名.(可以连接外网, 可以使用yum, wget等等.)
3. 配置防火墙，开启80、3306端口.(也可以直接关闭防火墙.)
	特别提示：如果这两条规则添加到防火墙配置的最后一行，导致防火墙启动失败，正确的应该是添加到默认的22端口后 。


		
	```
	vi /etc/sysconfig/iptables
	-A RH-Firewall-1-INPUT -m state --state NEW -m tcp -p tcp --dport 80 -j ACCEPT #允许80端口通过防火墙
	-A RH-Firewall-1-INPUT -m state --state NEW -m tcp -p tcp --dport 3306 -j ACCEPT #允许3306端口通过防火墙
	#最后重启防火墙使配置生效
	/etc/init.d/iptables restart 或 service iptables restart
	#查看防火墙状态.
	service iptables status
	```
			
4. 关闭SELinux

	```
	vi /etc/selinux/config
	#SELINUX=enforcing #注释掉
	#SELINUXTYPE=targeted #注释掉
	SELINUX=disabled #增加
	```
	
	`:wq!` #保存退出
	`setenforce 0 `#使配置立即生效.
	**注:**
	setenforce是Linux的selinux防火墙配置命令 
	执行setenforce 0 表示关闭selinux防火墙。
	setenforce命令是单词set（设置）和enforce(执行)连写.
	另一个命令getenforce可查看selinux的状态



===

## 系统约定

硬盘分区：20G(/boot: 200M /swap: 2048M /:剩余全部)
软件源代码包存放位置：/usr/local/src
源码包编译安装位置：/usr/local/软件名
数据库数据文件存储路径/usr/local/mysql/var

===

## 软件包下载

1. 下载nginx（目前稳定版）：http://nginx.org/download/nginx-1.4.4.tar.gz
2. 下载pcre （支持nginx伪静态）：pcre-8.39.tar.gz

	**注:** PCRE(Perl Compatible Regular Expressions)是一个Perl库，包括 perl 兼容的正则表达式库。
3. 下载MySQL：http://cdn.mysql.com/Downloads/MySQL-5.5/mysql-5.5.51.tar.gz
4. 下载php：http://cn2.php.net/distributions/php-5.5.7.tar.gz
5. 下载cmake（MySQL编译工具）：http://www.cmake.org/files/v2.8/cmake-2.8.12.1.tar.gz

	**注:** mysql  5.5以后的版本, 都需要用cmake安装
6. 下载libmcrypt（PHPlibmcrypt模块）：libmcrypt-2.5.7.tar.gz

	**注:** libmcrypt是加密算法扩展库
	
将以上软件包上传到/usr/local/src目录

## 安装编译工具及库文件

使用CentOS yum命令一键安装

```	
	yum install -y make apr* autoconf automake curl curl-devel gcc gcc-c++ gtk+-devel zlib-devel openssl openssl-devel pcre-devel gd kernel keyutils patch perl kernel-headers compat* cpp glibc libgomp libstdc++-devel keyutils-libs-devel libsepol-devel libselinux-devel krb5-devel libXpm* freetype freetype-devel freetype* fontconfig fontconfig-devel libjpeg* libpng* php-common php-gd gettext gettext-devel ncurses* libtool* libxml2 libxml2-devel patch policycoreutils bison
```

	Linux/centos Header V3 DSA signature: NOKEY, key ID
		错误解决方法: rpm --import /etc/pki/rpm-gpg/RPM* 
	Transaction Check Error:file /usr/lib64/libxcb-icccm.so.1.0.0 from install......
		错误解决方法: yum remove libxcb*, 然后再安装.
	这样, yum安装后就没有错误提示了.


## 软件安装篇

1. 安装cmake
	
	```
	mkdir /usr/local/cmake
	cd /usr/local/src
	tar zxvf cmake-2.8.12.1.tar.gz
	cd cmake-2.8.12.1
	./configure --prefix=/usr/local/cmake
	make #编译
	make install #安装
	```
	
	
	`vim /etc/profile` 在path路径中增加cmake执行文件路径, 最后一行添加:
		
	```
	export PATH=$PATH:/usr/local/cmake/bin
	source /etc/profile #使配置立即生效
	```


2. 安装pcre
	
	```
		cd /usr/local/src
		mkdir /usr/local/pcre #创建安装目录
		tar zxvf pcre-8.39.tar.gz
		cd pcre-8.39
		./configure --prefix=/usr/local/pcre #配置
		make && make install
	```
	
	**说明:**
	
	```
	make[1]: Entering directory /usr/local/src/pcre-8.39
	make[1]: Leaving directory /usr/local/src/pcre-8.39
	```
	
	这样的字段, 不是报错. 
	一般写makefile的时候，会cd到某个文件夹，再执行一些命令，这样的话，就会出现Leaving directory的信息

3. 安装libmcrypt
	
	```
	mkdir /usr/local/libmcrypt
	cd /usr/local/src
	tar zxvf libmcrypt-2.5.7.tar.gz #解压
	cd libmcrypt-2.5.7 #进入目录
	./configure #配置
	make #编译
	make install #安装
	```

4. 安装Mysql

	```
	groupadd mysql #添加mysql组
	useradd -g mysql mysql -s /bin/false #创建用户mysql并加入到mysql组，不允许mysql用户直接登录系统
	mkdir -p /usr/data/mysql/var #创建MySQL数据库存放目录
	chown -R mysql:mysql /usr/data/mysql/var #设置MySQL数据库目录权限
	cd /usr/local/src
	tar zxvf mysql-5.5.51.tar.gz #解压
	cd mysql-5.5.51
	cmake . -DCMAKE_INSTALL_PREFIX=/usr/local/mysql -DMYSQL_DATADIR=/usr/data/mysql/var -DSYSCONFDIR=/etc #配置
	make #编译
	make install #安装
	```

	```
	cd /usr/local/mysql
	cp ./support-files/my-huge.cnf /etc/my.cnf #拷贝配置文件
	```

	（注意：support-files, 这个目录中都是配置文件的模板. ）
	（注意：如果/etc目录下面默认有一个my.cnf，直接覆盖即可）
	
	```
	vi /etc/my.cnf #编辑配置文件,在 [mysqld] 部分增加
	datadir = /usr/data/mysql/var #添加MySQL数据库路径
	:wq退出编辑
	./scripts/mysql_install_db --user=mysql #生成mysql系统数据库
	```

	```
	cp ./support-files/mysql.server /etc/rc.d/init.d/mysqld #把Mysql加入系统启动
	chmod 755 /etc/init.d/mysqld #增加执行权限
	chkconfig mysqld on #加入开机启动
	vi /etc/rc.d/init.d/mysqld #编辑
	basedir = /usr/local/mysql #MySQL程序安装路径
	datadir = /usr/data/mysql/var #MySQl数据库存放目录
	service mysqld start #启动
	vi /etc/profile #把mysql服务加入系统环境变量：在最后添加下面这一行
	export PATH=$PATH:/usr/local/cmake/bin:/usr/local/mysql/bin
	source /etc/profile #使配置立即生效
	```

	```
	mkdir /var/lib/mysql #创建目录
	ln -s /tmp/mysql.sock /var/lib/mysql/mysql.sock #添加软链接
	```
	```
	mysql_secure_installation #设置Mysql密码，根据提示按Y 回车输入2次密码, (先输入当前密码, 一般为空.)
	/opt/local/mysql/bin/mysqladmin -u root -p password "123456" #或者直接修改密码
	```

	到此，mysql安装完成！



5. 安装 nginx

	```
	cd /usr/local/src
	groupadd www #添加www组
	useradd -g www www -s /bin/false #创建nginx运行账户www并加入到www组，不允许www用户直接登录系统, 分配一个不能登录的shell
	tar zxvf nginx-1.4.4.tar.gz
	cd nginx-1.4.4
	./configure --prefix=/usr/local/nginx --without-http_memcached_module --user=www --group=www --with-http_stub_status_module --with-openssl=/usr/ --with-pcre=/usr/local/src/pcre-8.39 #注意:--with-pcre=/usr/local/src/pcre-8.39指向的是源码包解压的路径，而不是安装的路径，否则会报错
	make
	make install
	```


	/usr/local/nginx/sbin/nginx #启动nginx
	设置nginx开启启动
	vi /etc/rc.d/init.d/nginx #编辑启动文件添加下面内容
	
	
	```
	#!/bin/bash
	# nginx Startup script for the Nginx HTTP Server
	# it is v.0.0.2 version.
	# chkconfig: - 85 15
	# description: Nginx is a high-performance web and proxy server.
	# It has a lot of features, but it's not for everyone.
	# processname: nginx
	# pidfile: /var/run/nginx.pid
	# config: /usr/local/nginx/conf/nginx.conf
	nginxd=/usr/local/nginx/sbin/nginx
	nginx_config=/usr/local/nginx/conf/nginx.conf
	nginx_pid=/usr/local/nginx/logs/nginx.pid
	RETVAL=0
	prog="nginx"
	# Source function library.
	. /etc/rc.d/init.d/functions
	# Source networking configuration.
	. /etc/sysconfig/network
	# Check that networking is up.
	[ ${NETWORKING} = "no" ] && exit 0
	[ -x $nginxd ] || exit 0
	# Start nginx daemons functions.
	start() {
	if [ -e $nginx_pid ];then
	echo "nginx already running...."
	exit 1
	fi
	echo -n $"Starting $prog: "
	daemon $nginxd -c ${nginx_config}
	RETVAL=$?
	echo
	[ $RETVAL = 0 ] && touch /var/lock/subsys/nginx
	return $RETVAL
	}
	# Stop nginx daemons functions.
	stop() {
	echo -n $"Stopping $prog: "
	killproc $nginxd
	RETVAL=$?
	echo
	[ $RETVAL = 0 ] && rm -f /var/lock/subsys/nginx /usr/local/nginx/logs/nginx.pid
	}
	reload() {
	echo -n $"Reloading $prog: "
	#kill -HUP `cat ${nginx_pid}`
	killproc $nginxd -HUP
	RETVAL=$?
	echo
	}
	# See how we were called.
	case "$1" in
	start)
	start
	;;
	stop)
	stop
	;;
	reload)
	reload
	;;
	restart)
	stop
	start
	;;
	status)
	status $prog
	RETVAL=$?
	;;
	*)
	echo $"Usage: $prog {start|stop|restart|reload|status|help}"
	exit 1
	esac
	exit $RETVAL
	```



	`:wq!` #保存退出
	`chmod 775 /etc/rc.d/init.d/nginx` #赋予文件执行权限
	`chkconfig nginx on` #设置开机启动
	`/etc/rc.d/init.d/nginx restart` #重新启动Nginx
	`service nginx restart`




6. 安装php

	```
	cd /usr/local/src
	tar -zvxf php-5.5.7.tar.gz
	cd php-5.5.7.
	./configure --prefix=/usr/local/php5 --with-config-file-path=/usr/local/php5/etc --with-mysql=/usr/local/mysql --with-mysql-sock=/tmp/mysql.sock --with-gd --with-iconv --with-zlib --enable-xml --enable-bcmath --enable-shmop --enable-sysvsem --enable-inline-optimization --enable-mbregex --enable-fpm --enable-mbstring --enable-ftp --enable-gd-native-ttf --with-openssl --enable-pcntl --enable-sockets --with-xmlrpc --enable-zip --enable-soap --without-pear --with-gettext --enable-session --with-mcrypt=/usr/local/libmcrypt --with-curl --with-jpeg-dir --with-freetype-dir
	
	make #编译
	make install #安装
	```
	*注:*
	
	```
	--with-mysql-sock:
		来源: http://www.2cto.com/database/201510/446700.html
		msyql 配置文件中有这个. socket = /tmp/mysql.sock
		在登录mysql的时候可以加上mysql.sock：
			[root@mysql ~]# mysql -u root -poldboy123 -S /tmp/mysql.sock
		Mysql有两种连接方式： 
		（1），TCP/IP 
		（2），socket 
		对mysql.sock来说，其作用是程序与mysqlserver处于同一台机器，发起本地连接时可用。 
		例如你无须定义连接host的具体IP地址，只要为空或localhost就可以。 
		在此种情况下，即使你改变mysql的外部port也是一样可能正常连接。 
		因为你在my.ini中或my.cnf中改变端口后，mysql.sock是随每一次 mysql server启动生成的。已经根据你在更改完my.cnf后重启mysql时重新生成了一次，信息已跟着变更。 
		那么对于外部连接，必须是要变更port才能连接的。 
		linux下安装mysql连接的时候经常回提示说找不到mysql.sock文件，解决办法很简单： 
		如果是新安装的mysql，提示找不到文件，就搜索下，指定正确的位置。 
		如果mysql.sock文件误删的话，就需要重启mysql服务，如果重启成功的话会在datadir目录下面生成mysql.sock 到时候指定即可。 
		如果还不行就选择用TCP连接方式连接就行了，其实windows下还支持管道连接方式。
	--with-gd
		gd库          
		如果 gd 是 rpm 安装的, 那就直接 --with-gd
		如果 gd 是编译安装的, 那么 --with-gd=/usr/local/gd
		后面接 gd 的安装路径
	--with-iconv
		作用: 使php能够使用iconv函数(编码转换函数)
		需要先安装libiconv
		--with-iconv-dir=/usr/local/libiconv
	--with-zlib
		压缩数据的作用
	--enable-shmop
		操作共享内存块, 类似于与memcache的作用

	报错: 
	configure: error: xml2-config not found. Please check your libxml2 installation.
	需要安装 libxml2 libxml2-devel

	configure: error: Cannot find OpenSSL's <evp.h>
	需要安装 openssl openssl-devel

	configure: error: Please reinstall the libcurl distribution - easy.h should be in <curl-dir>/include/curl/
	# yum -y install curl-devel

	configure: error: mcrypt.h not found. Please reinstall libmcrypt.
	之前安装了libmcrypt2.5.7, 所以在--with-mcrypt时需要指定路径. --with-mcrypt=/usr/local/libmcrypt
	如果在编译安装libmcrypt时, ./configure后面不写路径, 在编译安装php时, 直接写--with-mcrypt就可以了. 
```

	```
	cp php.ini-production /usr/local/php5/etc/php.ini #复制php配置文件到安装目录
	```
	
	说明: 
		php.ini-production	上线产品用的配置文件. 
		php.ini-development	开发环境用的配置文件.
		php.ini-production 拥有较高的安全性设定,则适合上线当产品使用

	```
	rm -rf /etc/php.ini #删除系统自带配置文件
	ln -s /usr/local/php5/etc/php.ini /etc/php.ini #添加软链接
	cp /usr/local/php5/etc/php-fpm.conf.default /usr/local/php5/etc/php-fpm.conf #拷贝模板文件为php-fpm配置文件

	vi /usr/local/php5/etc/php-fpm.conf #编辑
	user = www #设置php-fpm运行账号为www
	group = www #设置php-fpm运行组为www
	pid = run/php-fpm.pid #取消前面的分号
	```
	
	设置 php-fpm开机启动
	
	```
	cp /usr/local/src/php-5.5.7/sapi/fpm/init.d.php-fpm /etc/rc.d/init.d/php-fpm #拷贝php-fpm到启动目录
	chmod +x /etc/rc.d/init.d/php-fpm #添加执行权限
	chkconfig php-fpm on #设置开机启动
	vi /usr/local/php5/etc/php.ini #编辑配置文件
	找到：;date.timezone =
	修改为：date.timezone = PRC #设置时区
	找到：expose_php = On
	修改为：expose_php = OFF #禁止显示php版本的信息/
	找到：short_open_tag = Off
	修改为：short_open_tag = ON #支持php短标签
	```

7. 配置nginx支持php

	```
	vi /usr/local/nginx/conf/nginx.conf
	```
	
	修改/usr/local/nginx/conf/nginx.conf 配置文件,需做如下修改
	
	user www www; #首行user去掉注释,修改Nginx运行组为www www；必须与/usr/local/php/etc/php-fpm.conf中的user,group配置相同，否则php运行出错
	
	```
	user www www;
	worker_processes 1;
	events {
		worker_connections 1024;
	}
	
	http {
		include mime.types;
		default_type application/octet-stream;
		sendfile on;
		keepalive_timeout 65;
		server {
			listen 80;
			server_name  192.168.1.84;
			index index.php;
			root  /usr/local/nginx/html/;
			
			location / {
				autoindex on;
			}
			
			location ~ \.php {
				fastcgi_index index.php;
				fastcgi_pass 127.0.0.1:9000;
				include      fastcgi_params;
				set $path_info "";
				set $real_script_name $fastcgi_script_name;
				if ($fastcgi_script_name ~ "^(.+?\.php)(/.+)$") {
					set $real_script_name $1;
					set $path_info $2;
				}
				fastcgi_param SCRIPT_FILENAME $document_root/$real_script_name;
				fastcgi_param SCRIPT_NAME $real_script_name;
				fastcgi_param PATH_INFO $path_info;
			}
		}
	}
	```
	
	`/etc/init.d/nginx restart` #重启nginx
	

## 测试篇

```
	cd /usr/local/nginx/html/ #进入nginx默认网站根目录
	rm -rf /usr/local/nginx/html/* #删除默认测试页
	vi index.php #新建index.php文件
	phpinfo();
	?>
	:wq! #保存退出
	chown www.www /usr/local/nginx/html/ -R #设置目录所有者
	chmod 700 /usr/local/nginx/html/ -R #设置目录权限
```

*注:*
nginx启动了. 网站打不开. 打开nginx日志, 提示:
connect() failed (111: Connection refused) while connecting to upstream
大概意思是你没有启动或者配置php-fpm.其中“/usr/local/nginx/html”为网站根目录。
而我刚好是没有启动php-fpm,在终端运行“service php-fpm start”；
ok，正常运行了。

## 其它说明

服务器相关操作命令
	
```
service nginx restart #重启nginx

service mysqld restart #重启mysql

/usr/local/php/sbin/php-fpm #启动php-fpm

/etc/rc.d/init.d/php-fpm restart #重启php-fpm

/etc/rc.d/init.d/php-fpm stop #停止php-fpm

/etc/rc.d/init.d/php-fpm start #启动php-fpm

nginx默认站点目录是：/usr/local/nginx/html/

权限设置：chown www.www /usr/local/nginx/html/ -R

MySQL数据库目录是：/usr/local/mysql/var

权限设置：chown mysql.mysql -R /usr/local/mysql/var

```






