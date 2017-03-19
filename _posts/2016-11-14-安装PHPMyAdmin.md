---
layout: post
title: 安装PHPMyAdmin
categories: PHPMyAdmin
description: 安装PHPMyAdmin
keywords: PHPMyAdmin
---

# 安装PHPMyAdmin
去官网下载
[PHPMyAdmin官网](https://www.phpmyadmin.net/)

* 下载的应该是一个压缩包, 解压, 把加压后的文件放入PHP环境中的站点目录中. 我的站点目录是www. 
![](/images/posts/14791072031210.jpg)


* 进入站点中的phpmyadmin目录, 会看到一个示范用的配置文件. `config.sample.inc.php`, 复制一份, 把复制后的重命名, 改为`config.inc.php` .
![](/images/posts/14791072964208.jpg)

* 编辑此配置文件. 
我只修改了2个地方, 然后就可以使用了. 看注释就知道我是改的哪个地方了. 

```

/**
 * This is needed for cookie based authentication to encrypt password in
 * cookie. Needs to be 32 chars long.
 */
// $cfg['blowfish_secret'] = ''; /* YOU MUST FILL IN THIS FOR COOKIE AUTH! */
$cfg['blowfish_secret'] = 'aaaabbbbccccddddeeeeffffgggghhhh'; /* 改后的, 默认的是上一行. cookie加密用的, 必须32位字符 */

/**
 * Servers configuration
 */
$i = 0;

/**
 * First server
 */
$i++;
/* Authentication type */
$cfg['Servers'][$i]['auth_type'] = 'cookie';
/* Server parameters */
// $cfg['Servers'][$i]['host'] = 'localhost';
$cfg['Servers'][$i]['host'] = '192.168.1.186';	// 默认的是上一行, 这个就改为要连接的数据库的IP地址. 
$cfg['Servers'][$i]['connect_type'] = 'tcp';
$cfg['Servers'][$i]['compress'] = false;
$cfg['Servers'][$i]['AllowNoPassword'] = false;
```


* 最后, 在localhost中就可以看到phpmyadmin了, 进入, 输入数据库的用户和密码, 就可以使用了. 
![-w500](/images/posts/14791075850436.jpg)


![-w500](/images/posts/14791076774626.jpg)


![](/images/posts/14791076934215.jpg)


* 当然了, 方便的方法是给phpmyadmin搭建一个虚拟主机, 直接访问URL就可以进入phpmyadmin了. 

* 需改phpmyadmin的有效期
	每次登陆后, 过一会, 需要重新登陆, 很麻烦, 所以需要设置一下有效期. 
	解决方法如下：
修改php.ini，找到
session.gc_maxlifetime = 1440
将数值改大就行了，然后使之生效
试验了一下，结果不好使。
最终解决方案：
找到 `phpMyAdmin / libraries / config.default.php` 文件，打开，修改
`$cfg['LoginCookieValidity'] = 1440;`
将1440修改成更大的值即可。

注意：`$cfg['LoginCookieValidity']`的值不能大于php.ini里的`session.gc_maxlifetime`的值，否则phpmyadmin 里会出现下面的错误

```
“您的 PHP 配置参数 session.gc_maxlifetime (外链，英文) 短于您在 phpMyAdmin 中设置的 Cookies 有效期，因此您的登录会话有效期将会比您在 phpMyAdmin 中设置的时间要更短。”
```






