 ---
	layout: post
 title: 解决'RedisException' with message 'read error on connection'
	categories: php
	description: 解决'RedisException' with message 'read error on connection'
	keywords: php, redis
	---
	
	# 环境
centos 7

php sockets

redis

# 过程

最近一直困扰的问题得到了解决。

项目中利用redis做一个过度，起到从爬虫端到代理端的控制效果，对redis 

缓存进行一系列的操作，在实际使用中发现前面运行正常，过不了多久redis 

在php中报错: `PHP Fatal error:  Uncaught exception 'RedisException' with message 'read error on connection' in XXXX:35`

通过程序端设置set_time_limit(0), try/catch 捕获异常、服务器端查看日志，通过strace 跟踪进程执行时的系统调用等等也没有什么改善，问题依旧，一时感觉很是无解啊。

经过查找各方资料，发现是php.ini文件中的一个配置项导致：

## Conf修改
需要修改`php.ini`

`default_socket_timeout = 60 `

 由于redis扩展也是基于php 的socket方式实现，因此该参数值同样会起作用。
 
 找到了问题就比较好解决了：
       
 **1、直接修改php.ini，将其设置为我们想要的值（这个不推荐）**
 
 **2、在我们的脚本中通过以下方式设置，这样就比较灵活，不对其他脚本产生影响**
 
## PHP代码 

`ini_set('default_socket_timeout', -1);  //不超时 `
 
