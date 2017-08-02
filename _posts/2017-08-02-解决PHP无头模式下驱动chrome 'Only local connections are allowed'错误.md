---
layout: post
title: 解决PHP无头模式下驱动chrome 'Only local connections are allowed'错误
categories: Linux
description: 之前在一台centos7 服务器上搭建好了php + selenium + chrome 的爬虫环境, 但是由于服务器到期不得不需要重新部署一台环境爬虫环境, 突然发现selenium 驱动chrome 中有问题
keywords: chrome, php, selenium, Only local connections are allowed
---

# 起因

之前在一台centos7 服务器上搭建好了php + selenium + chrome 的

爬虫环境，但是由于服务器到期不得不需要重新部署一台环境爬虫环境，突然发

现selenium 驱动chrome 中有问题。

```bash

Starting ChromeDriver 2.30.477691 (6ee44a7247c639c0703f291d320bdf05c1531b57) on port 23129
Only local connections are allowed
```
在这个地方就停住了。

由于之前那台服务器环境装过太多东西了，原本无GUI还给装过桌面环境，一时

也想不起来具体需要哪一个组件。大家可以看下这篇博客具体讲了centos 下

搭建php-selenium + php-phantomjs + phantomjs +phpspider。

由于phantomjs 已经不维护了，还是把phantomjs 升级成chrome吧!

# 参考

centos7服务器无GUI情况下安装使用Xvfb、selenium、chrome和selenium-server: http://blog.csdn.net/xds2ml/article/details/52982748

在CentOS 7环境下安装chrome浏览器:http://www.cnblogs.com/fengbohello/p/4871445.html

# 安装Xvfb

yum install -y Xvfb

yum install -y libXfont

yum install xorg-x11-fonts*

这块安装在bash下没有问题，在zsh 下会报`zsh: no matches found:`错.

可以看下这篇文章:https://tonydeng.github.io/2014/12/09/zsh-scp-wildcard/

# 测试

## 新建php脚本

```php
<?php
namespace Facebook\WebDriver;

use Facebook\WebDriver\Remote\DesiredCapabilities;
use Facebook\WebDriver\Remote\RemoteWebDriver;
use Facebook\WebDriver\Remote\WebDriverCapabilityType;
use Facebook\WebDriver\Chrome\ChromeOptions;

require_once('vendor/autoload.php');


$options = new ChromeOptions();
$options->addArguments(array(
  'headless',
));

$capabilities = DesiredCapabilities::chrome();


$capabilities->setCapability(ChromeOptions::CAPABILITY, $options);


//$capabilities->setCapability(
//    'chrome.cli.args',
//    ['--proxy-auth='.getenv('WONDERPROXY_USER').':'.getenv('WONDERPROXY_PASS')]
//);

$driver = RemoteWebDriver::create(
    'http://localhost:4444/wd/hub',
    $capabilities
);

$driver->get("http://httpbin.org/ip");
$title = $driver->getPageSource();

echo $title."\n";
$driver->close();
$driver->quit();

die;
```

## 输出结果
```
<html xmlns="http://www.w3.org/1999/xhtml"><head></head><body><pre style="word-wrap: break-word; white-space: pre-wrap;">{
  "origin": "114.242.25.227"
}
</pre></body></html>

```

OK selenium 中也正常了.

```bash

Starting ChromeDriver 2.30.477691 (6ee44a7247c639c0703f291d320bdf05c1531b57) on port 23129
Only local connections are allowed.
15:36:56.399 INFO - Detected dialect: OSS
15:36:56.423 INFO - Done: [new session: Capabilities [{browserName=chrome, chromeOptions={args=[headless], binary=}, platform=ANY}]]
15:36:56.435 INFO - Executing: [get: http://httpbin.org/ip])
15:36:57.013 INFO - Done: [get: http://httpbin.org/ip]
15:36:57.017 INFO - Executing: [get page source])
15:36:57.026 INFO - Done: [get page source]
15:36:57.029 INFO - Executing: [close window])
15:36:57.135 INFO - Done: [close window]
15:36:57.138 INFO - Executing: [delete session: f980f3f3-8a93-44e4-81da-f32ecdf117db])
15:36:57.155 INFO - Done: [delete session: f980f3f3-8a93-44e4-81da-f32ecdf117db]

```


如果是在有桌面环境的centos 下可以忽略，不会出现这个问题.


