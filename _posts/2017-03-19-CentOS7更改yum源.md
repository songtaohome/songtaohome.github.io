---
layout: post
title: CeontOS7 更改yum源 
categories: Linux
description: CeontOS7 更改yum源的详细方法, 以及注意事项. 亲测ok. 
keywords: yum, CeontOS
---


# centos7 更改yum源

进入yum源的目录
`cd /etc/yum.repos.d`

先装上wget. 
`sudo yum -y install wget`

把原来的基础源给备份一下啊. 以防出现问题. 
`mv CentOS-Base.repo CentOS-Base.repo.bak`

```
[root@CentOS7 local]# cd /etc/yum.repos.d
[root@CentOS7 yum.repos.d]# ll
总用量 28
-rw-r--r--. 1 root root 1664 11月 30 02:12 CentOS-Base.repo
-rw-r--r--. 1 root root 1309 11月 30 02:12 CentOS-CR.repo
-rw-r--r--. 1 root root  649 11月 30 02:12 CentOS-Debuginfo.repo
-rw-r--r--. 1 root root  314 11月 30 02:12 CentOS-fasttrack.repo
-rw-r--r--. 1 root root  630 11月 30 02:12 CentOS-Media.repo
-rw-r--r--. 1 root root 1331 11月 30 02:12 CentOS-Sources.repo
-rw-r--r--. 1 root root 2893 11月 30 02:12 CentOS-Vault.repo
[root@CentOS7 yum.repos.d]# mv CentOS-Base.repo CentOS-Base.repo.bak
[root@CentOS7 yum.repos.d]# ll
总用量 28
-rw-r--r--. 1 root root 1664 11月 30 02:12 CentOS-Base.repo.bak
-rw-r--r--. 1 root root 1309 11月 30 02:12 CentOS-CR.repo
-rw-r--r--. 1 root root  649 11月 30 02:12 CentOS-Debuginfo.repo
-rw-r--r--. 1 root root  314 11月 30 02:12 CentOS-fasttrack.repo
-rw-r--r--. 1 root root  630 11月 30 02:12 CentOS-Media.repo
-rw-r--r--. 1 root root 1331 11月 30 02:12 CentOS-Sources.repo
-rw-r--r--. 1 root root 2893 11月 30 02:12 CentOS-Vault.repo
```

下载阿里的yum源
`wget -O CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo`

或者163yum源
`wget -O CentOS-Base.repo http://mirrors.163.com/.help/CentOS7-Base-163.repo`

下载完成后, 就变成了基础源. 

```
[root@CentOS7 yum.repos.d]# ll
总用量 32
-rw-r--r--. 1 root root 2573 5月  15 2015 CentOS-Base.repo
-rw-r--r--. 1 root root 1664 11月 30 02:12 CentOS-Base.repo.bak
-rw-r--r--. 1 root root 1309 11月 30 02:12 CentOS-CR.repo
-rw-r--r--. 1 root root  649 11月 30 02:12 CentOS-Debuginfo.repo
-rw-r--r--. 1 root root  314 11月 30 02:12 CentOS-fasttrack.repo
-rw-r--r--. 1 root root  630 11月 30 02:12 CentOS-Media.repo
-rw-r--r--. 1 root root 1331 11月 30 02:12 CentOS-Sources.repo
-rw-r--r--. 1 root root 2893 11月 30 02:12 CentOS-Vault.repo
```

再下载epel repo源源. 
`wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo`

```
[root@CentOS7 yum.repos.d]# ll
总用量 36
-rw-r--r--. 1 root root 2573 5月  15 2015 CentOS-Base.repo
-rw-r--r--. 1 root root 1664 11月 30 02:12 CentOS-Base.repo.bak
-rw-r--r--. 1 root root 1309 11月 30 02:12 CentOS-CR.repo
-rw-r--r--. 1 root root  649 11月 30 02:12 CentOS-Debuginfo.repo
-rw-r--r--. 1 root root  314 11月 30 02:12 CentOS-fasttrack.repo
-rw-r--r--. 1 root root  630 11月 30 02:12 CentOS-Media.repo
-rw-r--r--. 1 root root 1331 11月 30 02:12 CentOS-Sources.repo
-rw-r--r--. 1 root root 2893 11月 30 02:12 CentOS-Vault.repo
-rw-r--r--. 1 root root 1084 5月  15 2015 epel.repo
```

清理缓存

```
[root@CentOS7 yum.repos.d]# yum clean all
已加载插件：fastestmirror
正在清理软件源： base epel extras updates
Cleaning up everything
Cleaning up list of fastest mirrors
```

生成缓存

```
[root@CentOS7 yum.repos.d]# yum makecache
已加载插件：fastestmirror
http://mirrors.aliyuncs.com/centos/7/os/x86_64/repodata/repomd.xml: [Errno 12] Timeout on http://mirrors.aliyuncs.com/centos/7/os/x86_64/repodata/repomd.xml: (28, 'Connection timed out after 30001 milliseconds')
正在尝试其它镜像。
base                                                                                                                                                        | 3.6 kB  00:00:00
epel                                                                                                                                                        | 4.3 kB  00:00:00
extras                                                                                                                                                      | 3.4 kB  00:00:00
updates                                                                                                                                                     | 3.4 kB  00:00:00
(1/17): base/7/x86_64/filelists_db                                                                                                                          | 6.6 MB  00:00:15
(2/17): base/7/x86_64/group_gz                                                                                                                              | 155 kB  00:00:16
(3/17): base/7/x86_64/other_db                                                                                                                              | 2.4 MB  00:00:12
(4/17): epel/x86_64/group_gz                                                                                                                                | 170 kB  00:00:16
(5/17): base/7/x86_64/primary_db                                                                                                                            | 5.6 MB  00:00:18
(6/17): epel/x86_64/updateinfo                                                                                                                              | 753 kB  00:00:06
(7/17): epel/x86_64/filelists_db                                                                                                                            | 7.6 MB  00:00:37
(8/17): epel/x86_64/primary_db                                                                                                                              | 4.6 MB  00:00:14
(9/17): epel/x86_64/other_db                                                                                                                                | 2.1 MB  00:00:04
(10/17): extras/7/x86_64/prestodelta                                                                                                                        | 117 kB  00:00:06
(11/17): extras/7/x86_64/primary_db                                                                                                                         | 139 kB  00:00:00
(12/17): extras/7/x86_64/other_db                                                                                                                           | 545 kB  00:00:00
(13/17): updates/7/x86_64/prestodelta                                                                                                                       | 430 kB  00:00:06
(14/17): extras/7/x86_64/filelists_db                                                                                                                       | 596 kB  00:00:17
(15/17): updates/7/x86_64/primary_db                                                                                                                        | 3.8 MB  00:00:06
(16/17): updates/7/x86_64/filelists_db                                                                                                                      | 2.2 MB  00:00:23
(17/17): updates/7/x86_64/other_db                                                                                                                          |  44 MB  00:01:16
Determining fastest mirrors
 * base: mirrors.aliyun.com
 * epel: mirrors.aliyun.com
 * extras: mirrors.aliyun.com
 * updates: mirrors.aliyun.com
元数据缓存已建立
[root@CentOS7 yum.repos.d]#
```



**提示**
更改完yum源以后, 使用yum安装软件,会报错. 
有warnning, 如下

```
compat-poppler022-qt-0.22.5-4. FAILED                                          [==                                                               ] 616 kB/s |  11 MB  00:08:36 ETA
http://mirrors.163.com/centos/7/os/x86_64/Packages/compat-poppler022-qt-0.22.5-4.el7.x86_64.rpm: [Errno -1] 软件包与预期下载的不符。建议：运行 yum --enablerepo=base clean metadata
正在尝试其它镜像
```

然后就按照提示, 取消了yum, 执行`yum --enablerepo=base clean metadata`, 再次执行yum. 

还是报这个错, 上网搜索了一下答案, 说更改完yum源, 重启就好了. 我恢复快照, 试了一下, 果然, 就不会再出现这个问题了. 所以, 更改完yum源后, 需要`reboot`.

---

