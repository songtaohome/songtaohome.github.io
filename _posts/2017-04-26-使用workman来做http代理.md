---
layout: post
title: 2017-04-26-使用workman来做http代理.md
categories: Linux, PHP
description: 使用workman来做http代理
keywords: php,workman,代理,http,切换ip 
---

# 前言

工作中有块业务代码需要使用到爬虫，但是爬虫一直有个问题需要解决。一直

会遇到robot check。我们之前做的是在框架中切换http头信息除cookie 

等。效果明显，可后面想想能不能切换IP，做一个属于自己的代理。

# 方案

 用到的工具  redis, php-selenium, workman, 
 
# 参考

workman手册 : http://doc3.workerman.net/install/requirement.html

WonderProxy: https://wonderproxy.com/docs/developers/guides/globalize-your-testing-with-selenium

# 安装说明:

## 绑定本机网卡IP
  
  首先, 一般服务上一块网卡上有一个IP, 由于我们需要使用代理来切
  
  换IP，就需要在网卡上绑定多个IP。
  
  `ip addr add 2607:5300:60:3720::64 dev eth0`
  
## 查看ip 绑定是否生效

	`ip addr show eth0`
    
![](https://tao007.oss-cn-shanghai.aliyuncs.com/%E5%9B%BE%E7%89%87/QQ%E6%88%AA%E5%9B%BE20170426145924.bmp)
  
  也可以写一个脚本来循环绑定多个ip

## 删除绑定的ip 
  
   `ip addr del 2607:5300:60:3720::64 dev eth0`
  
  
## 下载workman 

git clone https://github.com/walkor/Workerman.git

## workman 环境检查

一般是在php.ini 中检查socket 是否被禁用

Linux用户可以运行以下脚本检查本地环境是否满足WorkerMan要求

curl -Ss http://www.workerman.net/check.php | php

如果脚本中全部提示ok，则代表满足WorkerMan运行环境

（注意：检测脚本中没有检测event扩展或者libevent扩展，如果并发连接数大于1024建议安装event扩展或者libevent扩展，安装方法参见下一节）


# 代码

```

$worker->onMessage = function($connection, $buffer)
{

    // Parse http header.
    list($method, $addr, $http_version) = explode(' ', $buffer);
    $url_data = parse_url($addr);
    $addr = !isset($url_data['port']) ? "{$url_data['host']}:80" : "{$url_data['host']}:{$url_data['port']}";


    $data = switchIp();
    //$data = array();
    if(is_array($data) && !empty($data)){
        $ip = $data['use_ip'];
        $context = (array(
            'socket' => array(
                'bindto' => $ip,
            ),
        ));
        // Async TCP connection.


        $remote_connection = new AsyncTcpConnection("tcp://$addr",$context);

    }else{
        $remote_connection = new AsyncTcpConnection("tcp://$addr");

    }
    // CONNECT.
    if ($method !== 'CONNECT') {
        $remote_connection->send($buffer);
        // POST GET PUT DELETE etc.
    } else {
        $connection->send("HTTP/1.1 200 Connection Established\r\n\r\n");
    }
    // Pipe.


    $remote_connection ->pipe($connection);
    $connection->pipe($remote_connection);

    $remote_connection->connect();

};

```
**问题说明:** 使用redis 可能会遇到 `RedisException' with message 'read error on connection`

看我的这篇博客:https://www.minsongtao.com/2017/04/11/%E8%A7%A3%E5%86%B3'RedisException'-with-message-read-error-on-connection/

# 测试

## 没有代理:

`curl http://v6.ipv6-test.com/api/myip.php`

![](https://tao007.oss-cn-shanghai.aliyuncs.com/%E5%9B%BE%E7%89%87/meiyoudaili.bmp)

## 开启代理之后:

`curl http://v6.ipv6-test.com/api/myip.php --proxy localhost:8081`

![](http://tao007.oss-cn-shanghai.aliyuncs.com/%E5%9B%BE%E7%89%87/%E5%BC%80%E5%90%AF%E4%BB%A3%E7%90%86.bmp)

## 总结 

workman 中AsyncTcpConnection是通过异步连接来向代理客户端回传数

据的，在特殊的需要非阻塞 多线程的情况下会出现卡住的问题，会造成客户

端卡住，请求超时的情况。





