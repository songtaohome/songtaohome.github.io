---
layout: post
title: Apache的配置文件总结, 是 linux/mac 下的配置
categories: Linux
description: apache的配置文件总结, 是 linux/mac 下的配置
keywords: Apache, Linux
---

我是用的 mac 的 brew自动安装的 Apache, apache 在 mac 中是叫httpd. 
安装后, Apache 的配置文件和资源文件是在不同的目录当中.
配置文件一般是在 /usr/local/etc/apache2/2.4/httpd.conf
apache的安装目录是在 /usr/local/Cellar/httpd24/2.4.18


## 针对主机环境的基本配置 

```
ServerRoot  # apache主目录
   # 这个是就是 apache 的安装目录, 不是配置文件的目录.  下面配置的路径, 如果用的是相对路径, 都是把这个当做根目录.
Listen   # 监听端口
   # 监听的端口, 可以写 IP:PORT, 或者直接写端口, Apache 的默认端口是80, 所以我这里就用的80
LoadModule  # 加载的相关模块
   # 后面有详细的解释
User   Group   # 用户和组
ServerAdmin  # 管理员邮箱
ServerName  # 服务器名
   # （没有域名解析时，使用临时解析。不开启）, 反向解析不能返回服务器名称的话, 就设置成 IP 地址
ErrorLog "logs/error_log # 错误日志
CustomLog "logs/access_log" common  # 正确访问日志
DirectoryIndex index.html index.php  # 默认网页文件名,优先级顺序
Include  etc/extra/httpd-vhosts.conf  # 子配置文件中内容也会加载生效 
```


## 主页目录及权限

```
# 站点的位置
DocumentRoot "/Users/admin/websides"  

# 权限
<Directory "/Users/admin/websides">
   # Indexes 就是控制 网站是否显示目录的开关. 比如: 如果index.php不存在就会显示网站下面的所有目录了, 当然正常情况下是关闭的indexs的。
   Options Indexes FollowSymLinks   
       # 选项, 可以有多个, 用空格分隔        
       # None：没有任何额外权限       
       # All： 所有权限       
       # Indexes：浏览权限（当此目录下没有默认网页文件时，显示目录内容）       
       # FollowSymLinks：准许软连接到其他目录


   # 开启rewrite模块对 URL 重写时, 会用到. 否则就是 None, 开启的话, 会找.htacess文件作为配置文件
   # 如果配置文件中, 修改了默认的重写文件名, 则.htaccess改成对应的名称. 如:
   # AccessFileName .cofig, 如果开启了重写的模块, 就应该在入口文件的同级目录下, 把重写配置文件的名称改为.config
   AllowOverride None  
       # 定义是否允许目录下.htaccess文件中的权限生效
       # None：.htaccess中权限不生效
       # All：文件中所有权限都生效
       # AuthConfig：文件中，只有网页认证的权限生效。


   # 访问控制列表
   Require all granted  
       # 定义此目录的允许访问权限  
       # 例子1： 仅允许IP为192.168.1.1的主机访问 
       <RequireAll>  
           Require all  denied  
           Require ip 192.168.1.1  
       </RequireAll>  
       # 例子2. 仅允许192.168.1.0/24网络的主机访问 
       <RequireAll>  
           Require  all  denied  
           Require ip 192.168.1.0/24  
       </RequireAll>  
       # 例子3.禁止192.168.1.2的主机访问,其他的都允许访问, 
       <RequireAll>  
           Require all  granted  
           Require not ip 192.168.1.2  
       </RequireAll>  
       # 例子4.允许所有访问
       Require all  granted   
       # 可以不写在<RequireAll>。。。</RequireAll>中
       # 例子5.拒绝所有访问
       Require all  denied    
       # 可以不写在<RequireAll>。。。</RequireAll>中
       </Directory>
```

## 目录别名

用途: 扩展网站目录，增加服务器，使用二级域名，使用目录别名
这个不是在 httpd.conf 中配置, 而是在扩展配置文件中. 位置: httpd.conf 所在目录下的 extra 文件夹里
子配置文件名 (httpd.conf所在目录)/extra/httpd-autoindex.conf
例如:

```
Alias /test1/ "/Users/admin/websides/test2/"   
<Directory "/Users/admin/websides/test2/">  
    Options Indexes MultiViews  
    AllowOverride None  
    Require all granted  
</Directory>
```

说明: 访问 URL (127.0.0.1/test1/), 则会访问/Users/admin/websides/test2/里的文件. 

注意: Alias对于斜线的使用,如果别名末尾使用的斜线,则对应目录也要以斜线结束;如果别名末尾没有斜线,则对应目录也不需要.

注意: Aliais只会影响本地URL(127.0.0.1/test1/ test1的部分)的对应,它不会修改URL的主机名称部分

注意: 使用此功能, 必须开启 alias 的模块. 




## 常用子配置文件

```
httpd-autoindex.conf        apache系统别名
httpd-default.conf          线程控制     
httpd-info.conf             状态统计网页
httpd-languages.conf        语言编码
httpd-manual.conf           apache帮助文档     
httpd-mpm.conf              最大连接数
   MaxRequestWorkers      250 （默认worker MPM模块生效）
httpd-multilang-errordoc.conf  错页面 
httpd-ssl.conf              ssl安全套接字访问
httpd-userdir.conf          用户主目录配置
httpd-vhosts.conf           虚拟主机 
```

## 模块

```
LoadModule php5_module libexec/libphp5.so  
        注意: 模块载入的顺序很重要。没有专家的建议，不要修改.
        在终端使用命令 httpd -l 可以查看 apache 已经使用的模块.
        libexec/libphp5.so 这个是用的相对路径, 就是相对于 ServerRoot 设置的根目录. 当然, 这里也可以写绝对路径. 
        如果启动apache服务的时候, 报错, 没有对应的模块, 那么, 就去 php 的安装目录, 把*. so的文件给复制到 apache 的 libexec目录下. 
	    上面这个我需要特别注意, 因为这个是控制使用哪个版本的 php 的. 如果安装了不同版本的 php.
        例如, 安装了5.2的和5.6的版本, 想使用5.2版本的话, 需要使用php5.2 安装目录下的 libphp5.so ,把这个拷贝到 apache安装目录下的 libexec 文件夹内. 就可以了. 

<IfModule dir_module>  
    DirectoryIndex index.php index.tpl index.html  
</IfModule>
```
  
这个意思的是: 如果开启了 `dir_module` 这个模块, 那么就执行里面的设置. 
这个设置是很常用的, 意思是, 如果指定的 文件目录里, 有 `index.php index.tpl index.html`, 那么久自动加载此文件, 有先后的顺序. 
使用这个模块的前提是: 开启了 `LoadModule dir_module libexec/mod_dir.so` , 也就是把注释取消掉. 

6 待续...



