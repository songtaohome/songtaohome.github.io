---
layout: post
title: CentOS7下安装lnmp (CentOS7.3.1661, Nginx1.8.1, MySQL5.6.16, PHP5.6.30)
categories: Linux
description: CentOS7下安装lnmp (CentOS7.3.1661, Nginx1.8.1, MySQL5.6.16, PHP5.6.30)
keywords: LNMP, CentOS, Linux
---

# CentOS7下安装lnmp (CentOS7.3.1661, Nginx1.8.1, MySQL5.6.16, PHP5.6.30)

**部署环境**: 
>
Mac PD 虚拟机
1G RAM
16G ROM
CentOS 7.3.1661 minimal X86 64 安装
更改yum源为163源


**安装位置说明**
>
未解压的源码包位置: `/usr/local/src`
解压后的源码包位置: `/usr/loca/src/softname`
安装位置: `/usr/local/softname`

**版本说明**
>
CentOS 7.3.1661
Nginx 1.8.1
MySQL 5.6.35
PHP 5.6.30
redis 3.2.8
memcached 1.4.35
other software : yum安装

---

### 知识科普

安装包分为**开发版**本和**稳定版本（GA）**，开发版本拥有最新的特性，但是并不稳定，也没有完全经过测试，可能存在严重的bug，而稳定版本是经过了长时间的测试，消除了具有已知的bug，其稳定性和安全性都得到一定的保障。  
  
对于一个mysql的版本号如：mysql-5.6.1-m1，这个版本号意味着什么呢？

1. **对于5.6.1的解释**：第一个数字5代表了文件格式，第二个数字6代表了发行级别，第三个数字1代表了版本号。更新幅度较小时，最后的数字会增加，出现了重大特性更新时，第二个数字会增加，文件格式改变时，第一个数字会增加.
2. **对于m1的解释**：这是用来表明这个mysql版本的稳定性级别的，如果没有这个后缀，那么这个版本就是一个稳定版（GA）；如果这个后缀是mN（例如m1，m2）格式，表明了这个版本加入了一些经过彻底测试的新特性，可以认为这是一个试生产的模具；如果这个后缀是rc，表明了这是一个候选版本，已经修改了已知的重要bug，但是没有经过足够长时间的使用来确认所有的bug已经被修复。

这些规定不知识针对于mysql, 别的安装包也是这样的规则. 如: 
`cmake-3.8.0-rc2.tar.gz`
这个是现在最新的版本. 也是遵循上述的规则的. 

---

### yum安装必备包和依赖包

```
yum install -y yum install -y make apr* autoconf automake curl curl-devel gcc gcc-c++ gtk+-devel zlib-devel openssl openssl-devel pcre-devel gd kernel keyutils patch perl kernel-headers compat* cpp glibc libgomp libstdc++-devel keyutils-libs-devel libsepol-devel libselinux-devel krb5-devel libXpm* freetype freetype-devel freetype* fontconfig fontconfig-devel libjpeg* libpng* php-common php-gd gettext gettext-devel ncurses* libtool* libxml2 libxml2-devel patch policycoreutils bison cmake libmcrypt
```



---
## 安装mysql5.6

#### 删除系统自带的mariadb

CentOs7版本默认情况下安装了mariadb-libs
**注**: (网上查询, 说必须先卸载才可以继续安装MySQL, 测试了, 不卸载也可以装的. 但是为了保险起见, 还是卸载了吧.)

首先查询一下, 是否有安装了mysql

```
[root@CentOS7 ~]# rpm -qa | grep mysql
apr-util-mysql-1.5.2-6.el7.x86_64
[root@CentOS7 ~]#
```

再查一下是否安装了mariadb

```
[root@CentOS7 mysql]# rpm -qa | grep -i mariadb-libs
mariadb-libs-5.5.52-1.el7.x86_64
[root@CentOS7 mysql]#
```

结果, 装了. 那么就卸载吧. 

```
[root@CentOS7 mysql]# yum remove mariadb-libs-5.5.52-1.el7.x86_64
```

```
移除  1 软件包 (+2 依赖软件包)

安装大小：17 M
是否继续？[y/N]：y
Downloading packages:
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  正在删除    : 2:postfix-2.10.1-6.el7.x86_64                                                                                                                                                           1/3
  正在删除    : apr-util-mysql-1.5.2-6.el7.x86_64                                                                                                                                                       2/3
  正在删除    : 1:mariadb-libs-5.5.52-1.el7.x86_64                                                                                                                                                      3/3
  验证中      : apr-util-mysql-1.5.2-6.el7.x86_64                                                                                                                                                       1/3
  验证中      : 2:postfix-2.10.1-6.el7.x86_64                                                                                                                                                           2/3
  验证中      : 1:mariadb-libs-5.5.52-1.el7.x86_64                                                                                                                                                      3/3

删除:
  mariadb-libs.x86_64 1:5.5.52-1.el7

作为依赖被删除:
  apr-util-mysql.x86_64 0:1.5.2-6.el7                                                                     postfix.x86_64 2:2.10.1-6.el7

完毕！
[root@CentOS7 mysql]#
```

在卸载的时候, 提示需要卸载2个依赖包. 这里得留意一下. 因为是先yum安装的依赖包, 所以这里卸载了. 可能依赖包就确实了. 为了保险起见. 再运行一次yum安装依赖包. 

```
yum install -y yum install -y make apr* autoconf automake curl curl-devel gcc gcc-c++ gtk+-devel zlib-devel openssl openssl-devel pcre-devel gd kernel keyutils patch perl kernel-headers compat* cpp glibc libgomp libstdc++-devel keyutils-libs-devel libsepol-devel libselinux-devel krb5-devel libXpm* freetype freetype-devel freetype* fontconfig fontconfig-devel libjpeg* libpng* php-common php-gd gettext gettext-devel ncurses* libtool* libxml2 libxml2-devel patch policycoreutils bison cmake
```

再查一下是否有安装过的mysql包

```
[root@CentOS7 mysql]# rpm -qa | grep -i mysql
[root@CentOS7 mysql]#
[root@CentOS7 mysql]# rpm -qa | grep -i mariadb-libs
[root@CentOS7 mysql]#
```

这里就证明已经卸载完毕了. 

再查找一下mysql的目录. 存在就都删除掉. 

```
[root@CentOS7 mysql]# find / -name mysql
/etc/selinux/targeted/active/modules/100/mysql
/usr/lib64/mysql
[root@CentOS7 mysql]#
```

删除掉查找出来的mysql目录

```
[root@CentOS7 mysql]# rm -rf /usr/lib64/mysql
[root@CentOS7 mysql]# rm -rf /etc/selinux/targeted/active/modules/100/mysql
```

再查一下是否还有mysql的目录

```
[root@CentOS7 mysql]# find / -name mysql
[root@CentOS7 mysql]#
```

ok, 删除干净了. 

---

### 创建mysql安装目录, 创建mysql用户和组

```
[root@CentOS7 mysql]# mkdir -p /usr/local/mysql/data
[root@CentOS7 mysql]# groupadd mysql
[root@CentOS7 mysql]# useradd -g mysql mysql -s /bin/nologin
[root@CentOS7 mysql]#
```

### 解压mysql5.6, 生成makefile文件, 编译安装

进入源码包目录

```
[root@CentOS7 mysql]# cd /usr/local/src/
[root@CentOS7 src]# ll
总用量 52960
-rw-r--r--. 1 root root   398312 3月  11 23:52 memcached-1.4.35.tar.gz
-rwxr-xr-x. 1 root root 32167628 3月  11 23:52 mysql-5.6.35.tar.gz
-rw-r--r--. 1 root root   833473 3月  11 23:52 nginx-1.8.1.tar.gz
-rw-r--r--. 1 root root 19274631 3月  11 23:52 php-5.6.30.tar.gz
-rwxr-xr-x. 1 root root  1547237 3月  11 23:52 redis-3.2.8.tar.gz
[root@CentOS7 src]#
```

解压mysql5.6, 进入解压后的目录

```
[root@CentOS7 src]# tar zxvf mysql-5.6.35.tar.gz
[root@CentOS7 src]# cd mysql-5.6.35
```

执行`cmake`, 生成makefile文件. 

```
cmake \
-DCMAKE_INSTALL_PREFIX=/usr/local/mysql \
-DMYSQL_DATADIR=/usr/local/mysql/data \
-DSYSCONFDIR=/etc \
-DWITH_MYISAM_STORAGE_ENGINE=1 \
-DWITH_INNOBASE_STORAGE_ENGINE=1 \
-DWITH_MEMORY_STORAGE_ENGINE=1 \
-DWITH_READLINE=1 \
-DMYSQL_UNIX_ADDR=/var/lib/mysql/mysql.sock \
-DMYSQL_TCP_PORT=3306 \
-DENABLED_LOCAL_INFILE=1 \
-DWITH_PARTITION_STORAGE_ENGINE=1 \
-DEXTRA_CHARSETS=all \
-DDEFAULT_CHARSET=utf8 \
-DDEFAULT_COLLATION=utf8_general_ci
```

解释一下各个配置的意思(网上找的):

```
# -DCMAKE_INSTALL_PREFIX=/usr/local/mysql56  \    #安装路径  
# -DMYSQL_DATADIR=/usr/local/mysql/data      \    #数据文件存放位置  
# -DSYSCONFDIR=/etc                         \    #my.cnf路径  
# -DWITH_MYISAM_STORAGE_ENGINE=1            \    #支持MyIASM引擎  
# -DWITH_INNOBASE_STORAGE_ENGINE=1          \    #支持InnoDB引擎  
# -DWITH_MEMORY_STORAGE_ENGINE=1            \    #支持Memory引擎  
# -DWITH_READLINE=1                         \    #快捷键功能(我没用过)  
# -DMYSQL_UNIX_ADDR=/tmp/mysqld.sock        \    #连接数据库socket路径  
# -DMYSQL_TCP_PORT=3306                     \    #端口  
# -DENABLED_LOCAL_INFILE=1                  \    #允许从本地导入数据  
# -DWITH_PARTITION_STORAGE_ENGINE=1         \    #安装支持数据库分区  
# -DEXTRA_CHARSETS=all                      \    #安装所有的字符集  
# -DDEFAULT_CHARSET=utf8                    \    #默认字符  
# -DDEFAULT_COLLATION=utf8_general_ci       \    #默认字符排序
```


完成后, 出现了warning

```
-- Configuring done
-- Generating done
CMake Warning:
  Manually-specified variables were not used by the project:

    WITH_MEMORY_STORAGE_ENGINE
    WITH_READLINE


-- Build files have been written to: /usr/local/src/mysql-5.6.35
[root@CentOS7 mysql-5.6.35]#
```

应该不影响.

继续编译安装.

```
[root@CentOS7 mysql-5.6.35]# make && make install
```

---


### 安装完毕后, 修改mysql的组和用户权限

先来看一下帮助的说明

```
[root@CentOS7 mysql]# scripts/mysql_install_db --help
Usage: scripts/mysql_install_db [OPTIONS]
  --basedir=path       The path to the MySQL installation directory.
  --builddir=path      If using --srcdir with out-of-directory builds, you
                       will need to set this to the location of the build
                       directory where built files reside.
  --cross-bootstrap    For internal use.  Used when building the MySQL system
                       tables on a different host than the target.
  --datadir=path       The path to the MySQL data directory.
                       If missing, the directory will be created, but its
                       parent directory must already exist and be writable.
  --defaults-extra-file=name
                       Read this file after the global files are read.
  --defaults-file=name Only read default options from the given file name.
  --force              Causes mysql_install_db to run even if DNS does not
                       work.  In that case, grant table entries that
                       normally use hostnames will use IP addresses.
  --help               Display this help and exit.
  --ldata=path         The path to the MySQL data directory. Same as --datadir.
  --no-defaults        Don't read default options from any option file.
  --keep-my-cnf        Don't try to create my.cnf based on template.
                       Useful for systems with working, updated my.cnf.
                       Deprecated, will be removed in future version.
  --random-passwords   Create and set a random password for all root accounts
                       and set the "password expired" flag,
                       also remove the anonymous accounts.
  --rpm                For internal use.  This option is used by RPM files
                       during the MySQL installation process.
  --skip-name-resolve  Use IP addresses rather than hostnames when creating
                       grant table entries.  This option can be useful if
                       your DNS does not work.
  --srcdir=path        The path to the MySQL source directory.  This option
                       uses the compiled binaries and support files within the
                       source tree, useful for if you don't want to install
                       MySQL yet and just want to create the system tables.
  --user=user_name     The login username to use for running mysqld.  Files
                       and directories created by mysqld will be owned by this
                       user.  You must be root to use this option.  By default
                       mysqld runs using your current login name and files and
                       directories that it creates will be owned by you.
Any other options are passed to the mysqld program.

[root@CentOS7 mysql]#
```


```
cd /usr/local/mysql
chown -R mysql:mysql .    #(这里最后是有个.的大家要注意# 为了安全安装完成后请修改权限给root用户)
scripts/mysql_install_db --user=mysql basedir=/usr/local/mysql --datadir=/usr/local/mysql/data     #(先进行这一步再做如下权限的修改)
chown -R root:mysql .     #(将权限设置给root用户，并设置给mysql组， 取消其他用户的读写执行权限，仅留给mysql "rx"读执行权限，其他用户无任何权限)
chown -R mysql:mysql ./data    #(数据库存放目录设置成mysql用户mysql组)
chmod -R ug+rwx  .     #(赋予读写执行权限，其他用户权限一律删除仅给mysql用户权限)
```


**配置myssql的配置文件**
下面的命令是将mysql的配置文件拷贝到`/etc`

```
cp support-files/my-default.cnf  /etc/my.cnf
```

修改my.cnf配置

```
vi /etc/my.cnf
```

设置mysql的配置文件. 以下是我在网上找的. 
参考: https://blog.imdst.com/mysql-5-6-pei-zhi-you-hua/

```
[client]
port = 3306  
socket = /tmp/mysql.sock

[mysql]
#这个配置段设置启动MySQL服务的条件；在这种情况下，no-auto-rehash确保这个服务启动得比较快。
no-auto-rehash

[mysqld]
user = mysql  
port = 3306  
socket = /tmp/mysql.sock  
basedir = /usr/local/mysql  
datadir = /usr/local/mysql/data/  
open_files_limit = 10240

back_log = 600  
#在MYSQL暂时停止响应新请求之前，短时间内的多少个请求可以被存在堆栈中。如果系统在短时间内有很多连接，则需要增大该参数的值，该参数值指定到来的TCP/IP连接的监听队列的大小。默认值80。

max_connections = 3000  
#MySQL允许最大的进程连接数，如果经常出现Too Many Connections的错误提示，则需要增大此值。默认151

max_connect_errors = 6000  
#设置每个主机的连接请求异常中断的最大次数，当超过该次数，MYSQL服务器将禁止host的连接请求，直到mysql服务器重启或通过flush hosts命令清空此host的相关信息。默认100

external-locking = FALSE  
#使用–skip-external-locking MySQL选项以避免外部锁定。该选项默认开启

max_allowed_packet = 32M  
#设置在网络传输中一次消息传输量的最大值。系统默认值 为4MB，最大值是1GB，必须设置1024的倍数。

#sort_buffer_size = 2M  
# Sort_Buffer_Size 是一个connection级参数，在每个connection（session）第一次需要使用这个buffer的时候，一次性分配设置的内存。
#Sort_Buffer_Size 并不是越大越好，由于是connection级的参数，过大的设置+高并发可能会耗尽系统内存资源。例如：500个连接将会消耗 500*sort_buffer_size(8M)=4G内存
#Sort_Buffer_Size 超过2KB的时候，就会使用mmap() 而不是 malloc() 来进行内存分配，导致效率降低。 系统默认2M，使用默认值即可

#join_buffer_size = 2M  
#用于表间关联缓存的大小，和sort_buffer_size一样，该参数对应的分配内存也是每个连接独享。系统默认2M，使用默认值即可

thread_cache_size = 300  
#默认38
# 服务器线程缓存这个值表示可以重新利用保存在缓存中线程的数量,当断开连接时如果缓存中还有空间,那么客户端的线程将被放到缓存中,如果线程重新被请求，那么请求将从缓存中读取,如果缓存中是空的或者是新的请求，那么这个线程将被重新创建,如果有很多新的线程，增加这个值可以改善系统性能.通过比较 Connections 和 Threads_created 状态的变量，可以看到这个变量的作用。设置规则如下：1GB 内存配置为8，2GB配置为16，3GB配置为32，4GB或更高内存，可配置更大。

#thread_concurrency = 8  
#系统默认为10，使用10先观察
# 设置thread_concurrency的值的正确与否, 对mysql的性能影响很大, 在多个cpu(或多核)的情况下，错误设置了thread_concurrency的值, 会导致mysql不能充分利用多cpu(或多核), 出现同一时刻只能一个cpu(或核)在工作的情况。thread_concurrency应设为CPU核数的2倍. 比如有一个双核的CPU, 那么thread_concurrency的应该为4; 2个双核的cpu, thread_concurrency的值应为8

query_cache_size = 64M  
#在MyISAM引擎优化中，这个参数也是一个重要的优化参数。但也爆露出来一些问题。机器的内存越来越大，习惯性把参数分配的值越来越大。这个参数加大后也引发了一系列问题。我们首先分析一下 query_cache_size的工作原理：一个SELECT查询在DB中工作后，DB会把该语句缓存下来，当同样的一个SQL再次来到DB里调用时，DB在该表没发生变化的情况下把结果从缓存中返回给Client。这里有一个关建点，就是DB在利用Query_cache工作时，要求该语句涉及的表在这段时间内没有发生变更。那如果该表在发生变更时，Query_cache里的数据又怎么处理呢？首先要把Query_cache和该表相关的语句全部置为失效，然后在写入更新。那么如果Query_cache非常大，该表的查询结构又比较多，查询语句失效也慢，一个更新或是Insert就会很慢，这样看到的就是Update或是Insert怎么这么慢了。所以在数据库写入量或是更新量也比较大的系统，该参数不适合分配过大。而且在高并发，写入量大的系统，建议把该功能禁掉。

query_cache_limit = 4M  
#指定单个查询能够使用的缓冲区大小，缺省为1M

query_cache_min_res_unit = 2k  
#默认是4KB，设置值大对大数据查询有好处，但如果你的查询都是小数据查询，就容易造成内存碎片和浪费
#查询缓存碎片率 = Qcache_free_blocks / Qcache_total_blocks * 100%
#如果查询缓存碎片率超过20%，可以用FLUSH QUERY CACHE整理缓存碎片，或者试试减小query_cache_min_res_unit，如果你的查询都是小数据量的话。
#查询缓存利用率 = (query_cache_size – Qcache_free_memory) / query_cache_size * 100%
#查询缓存利用率在25%以下的话说明query_cache_size设置的过大，可适当减小;查询缓存利用率在80%以上而且Qcache_lowmem_prunes > 50的话说明query_cache_size可能有点小，要不就是碎片太多。
#查询缓存命中率 = (Qcache_hits – Qcache_inserts) / Qcache_hits * 100%

#default-storage-engine = MyISAM
#default_table_type = InnoDB #开启失败

#thread_stack = 192K  
#设置MYSQL每个线程的堆栈大小，默认值足够大，可满足普通操作。可设置范围为128K至4GB，默认为256KB，使用默认观察

transaction_isolation = READ-COMMITTED  
# 设定默认的事务隔离级别.可用的级别如下:READ UNCOMMITTED-读未提交 READ COMMITTE-读已提交 REPEATABLE READ -可重复读 SERIALIZABLE -串行

tmp_table_size = 256M  
# tmp_table_size 的默认大小是 32M。如果一张临时表超出该大小，MySQL产生一个 The table tbl_name is full 形式的错误，如果你做很多高级 GROUP BY 查询，增加 tmp_table_size 值。如果超过该值，则会将临时表写入磁盘。
max_heap_table_size = 256M

expire_logs_days = 7  
key_buffer_size = 2048M  
#批定用于索引的缓冲区大小，增加它可以得到更好的索引处理性能，对于内存在4GB左右的服务器来说，该参数可设置为256MB或384MB。

read_buffer_size = 1M  
#默认128K
# MySql读入缓冲区大小。对表进行顺序扫描的请求将分配一个读入缓冲区，MySql会为它分配一段内存缓冲区。read_buffer_size变量控制这一缓冲区的大小。如果对表的顺序扫描请求非常频繁，并且你认为频繁扫描进行得太慢，可以通过增加该变量值以及内存缓冲区大小提高其性能。和sort_buffer_size一样，该参数对应的分配内存也是每个连接独享。

read_rnd_buffer_size = 16M  
# MySql的随机读（查询操作）缓冲区大小。当按任意顺序读取行时(例如，按照排序顺序)，将分配一个随机读缓存区。进行排序查询时，MySql会首先扫描一遍该缓冲，以避免磁盘搜索，提高查询速度，如果需要排序大量数据，可适当调高该值。但MySql会为每个客户连接发放该缓冲空间，所以应尽量适当设置该值，以避免内存开销过大。

bulk_insert_buffer_size = 64M  
#批量插入数据缓存大小，可以有效提高插入效率，默认为8M

myisam_sort_buffer_size = 128M  
# MyISAM表发生变化时重新排序所需的缓冲 默认8M

myisam_max_sort_file_size = 10G  
# MySQL重建索引时所允许的最大临时文件的大小 (当 REPAIR, ALTER TABLE 或者 LOAD DATA INFILE).
# 如果文件大小比此值更大,索引会通过键值缓冲创建(更慢)

#myisam_max_extra_sort_file_size = 10G 5.6无此值设置
#myisam_repair_threads = 1   默认为1
# 如果一个表拥有超过一个索引, MyISAM 可以通过并行排序使用超过一个线程去修复他们.
# 这对于拥有多个CPU以及大量内存情况的用户,是一个很好的选择.

myisam_recover  
#自动检查和修复没有适当关闭的 MyISAM 表
skip-name-resolve  
lower_case_table_names = 1  
server-id = 1

innodb_additional_mem_pool_size = 16M  
#这个参数用来设置 InnoDB 存储的数据目录信息和其它内部数据结构的内存池大小，类似于Oracle的library cache。这不是一个强制参数，可以被突破。

innodb_buffer_pool_size = 2048M  
# 这对Innodb表来说非常重要。Innodb相比MyISAM表对缓冲更为敏感。MyISAM可以在默认的 key_buffer_size 设置下运行的可以，然而Innodb在默认的 innodb_buffer_pool_size 设置下却跟蜗牛似的。由于Innodb把数据和索引都缓存起来，无需留给操作系统太多的内存，因此如果只需要用Innodb的话则可以设置它高达 70-80% 的可用内存。一些应用于 key_buffer 的规则有 — 如果你的数据量不大，并且不会暴增，那么无需把 innodb_buffer_pool_size 设置的太大了

#innodb_data_file_path = ibdata1:1024M:autoextend 设置过大导致报错，默认12M观察
#表空间文件 重要数据

#innodb_file_io_threads = 4   不明确，使用默认值
#文件IO的线程数，一般为 4，但是在 Windows 下，可以设置得较大。


innodb_thread_concurrency = 8  
#服务器有几个CPU就设置为几，建议用默认设置，一般为8.

innodb_flush_log_at_trx_commit = 2  
# 如果将此参数设置为1，将在每次提交事务后将日志写入磁盘。为提供性能，可以设置为0或2，但要承担在发生故障时丢失数据的风险。设置为0表示事务日志写入日志文件，而日志文件每秒刷新到磁盘一次。设置为2表示事务日志将在提交时写入日志，但日志文件每次刷新到磁盘一次。

#innodb_log_buffer_size = 16M   使用默认8M
#此参数确定些日志文件所用的内存大小，以M为单位。缓冲区更大能提高性能，但意外的故障将会丢失数据.MySQL开发人员建议设置为1－8M之间

#innodb_log_file_size = 128M  使用默认48M
#此参数确定数据日志文件的大小，以M为单位，更大的设置可以提高性能，但也会增加恢复故障数据库所需的时间

#innodb_log_files_in_group = 3   使用默认2
#为提高性能，MySQL可以以循环方式将日志文件写到多个文件。推荐设置为3M

#innodb_max_dirty_pages_pct = 90  使用默认75观察
#推荐阅读 http://www.taobaodba.com/html/221_innodb_max_dirty_pages_pct_checkpoint.html
# Buffer_Pool中Dirty_Page所占的数量，直接影响InnoDB的关闭时间。参数innodb_max_dirty_pages_pct 可以直接控制了Dirty_Page在Buffer_Pool中所占的比率，而且幸运的是innodb_max_dirty_pages_pct是可以动态改变的。所以，在关闭InnoDB之前先将innodb_max_dirty_pages_pct调小，强制数据块Flush一段时间，则能够大大缩短 MySQL关闭的时间。

innodb_lock_wait_timeout = 120  
#默认为50秒 
# InnoDB 有其内置的死锁检测机制，能导致未完成的事务回滚。但是，如果结合InnoDB使用MyISAM的lock tables 语句或第三方事务引擎,则InnoDB无法识别死锁。为消除这种可能性，可以将innodb_lock_wait_timeout设置为一个整数值，指示 MySQL在允许其他事务修改那些最终受事务回滚的数据之前要等待多长时间(秒数)

innodb_file_per_table = 0  
#默认为No
#独享表空间（关闭）

[mysqldump]
quick  
# max_allowed_packet = 32M

[mysqld_safe]
log-error=/data/mysql/mysql_oldboy.err  
pid-file=/data/mysql/mysqld.pid

sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES 
```

把mysqld加入系统启动, 设置权限, 开机启动. 

```
cp ./support-files/mysql.server /etc/rc.d/init.d/mysqld
chmod 755 /etc/init.d/mysqld
chkconfig mysqld on
```

编辑开机启动的mysqld文件. 

```
vi /etc/rc.d/init.d/mysqld
```

指定mysql安装目录和数据库文件目录

保存, 退出编辑

启动mysql

```
[root@iZ28cww0nf3Z mysql]# service mysqld start
Starting MySQL...                                          [  OK  ]
```

把mysql加入环境变量中. 

```
[root@iZ28cww0nf3Z mysql]# vi /etc/profile
```

在最面新增一行

```
export PATH=$PATH:/usr/local/mysql/bin
```

退出保存, 然后让新加的环境变量立即生效

```
[root@iZ28cww0nf3Z mysql]# source /etc/profile
```

添加mysql.sock的软链

```
[root@iZ28cww0nf3Z mysql]# mkdir /var/lib/mysql
[root@iZ28cww0nf3Z mysql]# ln -s /tmp/mysql.sock /var/lib/mysql/mysql.sock
```

## 安装ngnix

```
[root@CentOS7 ~]# groupadd www
[root@CentOS7 ~]# useradd -g www www -s /bin/nologin
[root@CentOS7 ~]# mkdir /usr/local/nginx
[root@CentOS7 ~]# cd /usr/local/src
[root@CentOS7 src]# ls
memcached-1.4.35.tar.gz  mysql-5.6.35  mysql-5.6.35.tar.gz  nginx-1.8.1.tar.gz  php-5.6.30.tar.gz  redis-3.2.8.tar.gz
[root@CentOS7 src]# tar zxvf nginx-1.8.1.tar.gz
```

其中./configure时是可以进行一些配置的，具体可以参考（如果你看英文手册没有什么压力的话可以直接 configure --help获取帮助）：

--prefix=PATH ： 指定nginx的安装目录。默认 /usr/local/nginx

--conf-path=PATH ： 设置nginx.conf配置文件的路径。nginx允许使用不同的配置文件启动，通过命令行中的-c选项。默认为prefix/conf/nginx.conf

--user=name： 设置nginx工作进程的用户。安装完成后，可以随时在nginx.conf配置文件更改user指令。默认的用户名是nobody。--group=name类似

--with-pcre ： 设置PCRE库的源码路径，如果已通过yum方式安装，使用--with-pcre自动找到库文件。使用--with-pcre=PATH时，需要从PCRE网站下载pcre库的源码（版本4.4 – 8.30）并解压，剩下的就交给Nginx的./configure和make来完成。perl正则表达式使用在location指令和 ngx_http_rewrite_module模块中。

--with-zlib=PATH ： 指定 zlib（版本1.1.3 – 1.2.5）的源码解压目录。在默认就启用的网络传输压缩模块ngx_http_gzip_module时需要使用zlib 。

--with-http_ssl_module ： 使用https协议模块。默认情况下，该模块没有被构建。前提是openssl与openssl-devel已安装

--with-http_stub_status_module ： 用来监控 Nginx 的当前状态

--with-http_realip_module ： 通过这个模块允许我们改变客户端请求头中客户端IP地址值(例如X-Real-IP 或 X-Forwarded-For)，意义在于能够使得后台服务器记录原始客户端的IP地址

--add-module=PATH ： 添加第三方外部模块，如nginx-sticky-module-ng或缓存模块。每次添加新的模块都要重新编译（Tengine可以在新加入module时无需重新编译）

不用这个. 

```
./configure \
--user=www \
--group=www \
--prefix=/usr/local/nginx \
--error-log-path=/var/log/nginx/error.log \
--http-log-path=/var/log/nginx/access.log \
--pid-path=/var/run/nginx.pid \
--with-http_stub_status_module \
--with-http_ssl_module \
--with-http_gzip_static_module \
--with-pcre \
--with-http_realip_module \
--with-http_sub_module
```

也不用这个. 

```
./configure \
--user=www \
--group=www \
--prefix=/usr/local/nginx \
--error-log-path=/var/log/nginx/error.log \
--http-log-path=/var/log/nginx/access.log \
--pid-path=/var/run/nginx.pid \
--with-http_stub_status_module \
--with-http_ssl_module \
--with-http_gzip_static_module \
--with-pcre \
--with-http_realip_module \
--with-http_sub_module
```

用这个. 其他的默认就行了. 

```
[root@iZ28cww0nf3Z nginx-1.4.4]# ./configure \
--prefix=/usr/local/nginx \
--user=www \
--group=www \
--with-http_stub_status_module \
--with-pcre
```

```
make && make install
```

安装完毕后, 启动nginx

```
[root@iZ28cww0nf3Z nginx-1.4.4]# /usr/local/nginx/sbin/nginx
```

以后每次都这么启动太麻烦, 设置开机自动启动. 
编辑文件:

```
[root@iZ28cww0nf3Z nginx-1.4.4]# vi /etc/rc.d/init.d/nginx
```

加入以下内容. 

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
nginx_pid=/var/run/nginx.pid
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

然后保存退出, 再设置一下该文件的权限.且加入开机启动. 

```
[root@iZ28cww0nf3Z nginx-1.4.4]# chmod 775 /etc/rc.d/init.d/nginx
[root@iZ28cww0nf3Z nginx-1.4.4]# chkconfig nginx on
```

现在可以使用命令了. 如: 
1. 重启nginx `service nginx restart`
2. 停止nginx `service nginx stop`
3. 启动nginx `service nginx start`

重启nginx

```
[root@iZ28cww0nf3Z nginx-1.4.4]# service nginx restart
Stopping nginx:                                            [  OK  ]
Starting nginx:                                            [  OK  ]
[root@iZ28cww0nf3Z nginx-1.4.4]# 
```




安装完毕. 

**疑问**
nginx进程的所属用户是nginx, 但是nginx的log文件是root, 为什么还能写入呢?




## 源码安装PHP

首先, 还是创建安装目录, 然后解压, 进入解压的目录, 配置编译选项

需要先安装`libmcrypt`, 解压, 安装. 

```
[root@iZ28cww0nf3Z php-5.5.7]# ./configure \
--prefix=/usr/local/php \
--with-config-file-path=/usr/local/php/etc \
--with-mysql=/usr/local/mysql \
--with-mysql-sock=/tmp/mysql.sock \
--with-gd \
--with-iconv \
--with-zlib \
--enable-xml \
--enable-bcmath \
--enable-shmop \
--enable-sysvsem \
--enable-inline-optimization \
--enable-mbregex \
--enable-fpm \
--enable-mbstring \
--enable-ftp \
--enable-gd-native-ttf \
--with-openssl \
--enable-pcntl \
--enable-sockets \
--with-xmlrpc \
--enable-zip \
--enable-soap \
--without-pear \
--with-gettext \
--enable-session \
--with-mcrypt=/usr/local/libmcrypt \
--with-curl \
--with-jpeg-dir \
--with-freetype-dir
```

出现报错信息:

```
configure: error: png.h not found.
```

解决方案: 

```
yum install libpng-devel
```

保险起见, 之前yum安装的再执行一遍. 

再次执行./configure 


ok, 这次没有报错. 

然后安装

```
[root@iZ28cww0nf3Z php-5.5.7]# make && make install
```

等了好久, 终于安装完毕了. 

开始配置php
复制php配置文件到安装目录, 删除系统自带配置文件, 然后添加软链接

```
[root@test php-5.6.30]# cp php.ini-production /usr/local/php/etc/php.ini
[root@test php-5.6.30]# rm -rf /etc/php.ini
[root@test php-5.6.30]# ln -s /usr/local/php/etc/php.ini /etc/php.ini
```

拷贝模板文件为php-fpm配置文件, 然后编辑

```
[root@test php-5.6.30]# cp /usr/local/php/etc/php-fpm.conf.default /usr/local/php/etc/php-fpm.conf
[root@test php-5.6.30]# vi /usr/local/php/etc/php-fpm.conf
```


找到user和group的配置位置
![-w618](/images/posts/14795348460790.jpg)
改为: 
![-w614](/images/posts/14795348991727.jpg)

再找到pid的配置位置
![-w608](/images/posts/14795350004961.jpg)
去掉前面的分号, 改为
![-w606](/images/posts/14795350421566.jpg)
配置完毕, 保存退出. 

设置 php-fpm开机启动
1. 拷贝php-fpm到启动目录
2. 添加执行权限
3. 设置开机启动

```
[root@test php-5.6.30]# cp /usr/local/src/php-5.6.30/sapi/fpm/init.d.php-fpm /etc/rc.d/init.d/php-fpm
[root@test php-5.6.30]# chmod +x /etc/rc.d/init.d/php-fpm
[root@test php-5.6.30]# chkconfig php-fpm on
```

编辑配置文件

**设置时区:**
找到：`;date.timezone =` , 修改为：`date.timezone = PRC `
**禁止显示php版本的信息:**
找到：`expose_php = On` , 修改为：`expose_php = OFF` 
**支持php短标签:**
找到：`short_open_tag = Off` , 修改为：`short_open_tag = ON`

## 配置nginx支持php

编辑nginx的配置文件

```
[root@iZ28cww0nf3Z php-5.5.7]# vi /usr/local/nginx/conf/nginx.conf
```

前面提到过, 我把`/usr/local/nginx/html`目录作为站点的根目录, 而且也给他加了www的组, 且附了权限. 

现在我想新建个目录`/data/www/blog`作为我的博客新站点目录.
赋予权限, 赋予组, 同理. 

我的配置

```
user  www www;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;

events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;

    server {
        listen       80;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            root   html;
            index  index.html index.htm index.php;
        }

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        #PHP 脚本请求全部转发到 FastCGI处理. 使用FastCGI默认配置.
        location ~ .php$ {
            fastcgi_pass 127.0.0.1:9000;
            fastcgi_index index.php;
            fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
            include fastcgi_params;
        }

    }

}
```

启动php-fpm

```
service php-fpm start
```


## LNMP常用命令总结

`service nginx restart` #重启nginx
`service nginx start` #启动nginx
`service nginx stop` #停止nginx

`service mysqld restart` #重启mysql

`/usr/local/php/sbin/php-fpm` #启动php-fpm
`/etc/rc.d/init.d/php-fpm restart` #重启php-fpm
`/etc/rc.d/init.d/php-fpm stop` #停止php-fpm
`/etc/rc.d/init.d/php-fpm start` #启动php-fpm

`service php-fpm start` #启动php-fpm
`service php-fpm restart` #重启php-fpm
`service php-fpm stop` #关闭php-fpm

nginx默认站点目录是：`/usr/local/nginx/html/`
权限设置：`chown www.www /usr/local/nginx/html/ -R`
MySQL数据库目录是：`/usr/local/mysql/var`
权限设置：`chown mysql.mysql -R /usr/local/mysql/var`

终于搞定了

![-w1187](/images/posts/14893359417056.jpg)




参考: 
http://blog.csdn.net/wplblog/article/details/52179299
http://www.linuxidc.com/Linux/2015-06/119354.htm








