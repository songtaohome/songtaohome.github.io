yout: post
title: 虚拟机下搭建CentOS, php-selenium,phantomjs 
categories: Linux
description: 虚拟机下搭建CentOS, php-selenium,phantomjs, 以及注意事项. 亲测ok. 
keywords: yum, CeontOS,php-selenium,phantomjs
---


# 方案
用到的模块: php-selenium + php-phantomjs + phantomjs + phpspider



# 参考
Selenium WebDriver set up with PHP - Selenium PHP traininig: https://www.youtube.com/watch?v=554KH7Ok1tA

facebook php-webdriver : https://github.com/facebook/php-webdriver https://github.com/Nearsoft/php-selenium-client http://facebook.github.io/php-webdriver/latest/index.html

selenium 文档: http://www.kancloud.cn/wangking/selenium/234398

phantomjs 文档: http://phantomjs.org/examples/index.html

思路整理: https://www.v2ex.com/t/324309

只用方法: https://phptrends.com/dig_in/php-webdriver http://blog.kelu.org/tech/2017/02/16/selenium-tutorial.html

书籍: https://www.amazon.com/Selenium-Webdriver-PHP-Beginners-Guide/dp/1540672972

python-selenium中文手册: http://selenium-python-zh.readthedocs.io/en/latest/index.html

新页面打开链接的方法: https://www.oschina.net/question/928852_86769

# 版本说明

* CentOS 7.3.1661 minimal
* php 5.6.28(yum安装的, 主要就用命令行)
* selenium-server-standalone-3.3.1.jar
* Composer 1.4.1
* phantomjs
* 项目目录 /root/test

# 安装说明
下载composer: curl -sS https://getcomposer.org/installer | php
```
[root@CentOS7 test]# ll
总用量 1796
-rwxr-xr-x. 1 root root 1836198 3月  20 12:33 composer.phar
[root@CentOS7 test]#
```
java jdk下载地址: http://www.oracle.com/technetwork/java/javase/downloads/index.html http://download.csdn.net/detail/tan3739/9678434

安装jdk
```
[root@CentOS7 test]# rpm -ivh jdk-8u111-linux-x64.rpm
准备中...                          ################################# [100%]
正在升级/安装...
   1:jdk1.8.0_111-2000:1.8.0_111-fcs  ################################# [100%]
Unpacking JAR files...
	tools.jar...
	plugin.jar...
	javaws.jar...
	deploy.jar...
	rt.jar...
	jsse.jar...
	charsets.jar...
	localedata.jar...
[root@CentOS7 test]#
```
是否安装成功, 查看版本
```
[root@CentOS7 test]# java -version
java version "1.8.0_111"
Java(TM) SE Runtime Environment (build 1.8.0_111-b14)
Java HotSpot(TM) 64-Bit Server VM (build 25.111-b14, mixed mode)
[root@CentOS7 test]#
```
运行
```
[root@CentOS7 test]# java -jar selenium-server-standalone-3.3.1.jar
14:52:47.065 INFO - Selenium build info: version: '3.3.1', revision: '5234b32'
14:52:47.066 INFO - Launching a standalone Selenium Server
2017-03-20 14:52:47.120:INFO::main: Logging initialized @321ms to org.seleniumhq.jetty9.util.log.StdErrLog
14:52:47.235 INFO - Driver provider org.openqa.selenium.ie.InternetExplorerDriver registration is skipped:
 registration capabilities Capabilities [{ensureCleanSession=true, browserName=internet explorer, version=, platform=WINDOWS}] does not match the current platform LINUX
14:52:47.235 INFO - Driver provider org.openqa.selenium.edge.EdgeDriver registration is skipped:
 registration capabilities Capabilities [{browserName=MicrosoftEdge, version=, platform=WINDOWS}] does not match the current platform LINUX
14:52:47.235 INFO - Driver class not found: com.opera.core.systems.OperaDriver
14:52:47.236 INFO - Driver provider com.opera.core.systems.OperaDriver registration is skipped:
Unable to create new instances on this machine.
14:52:47.236 INFO - Driver class not found: com.opera.core.systems.OperaDriver
14:52:47.236 INFO - Driver provider com.opera.core.systems.OperaDriver is not registered
14:52:47.237 INFO - Driver provider org.openqa.selenium.safari.SafariDriver registration is skipped:
 registration capabilities Capabilities [{browserName=safari, version=, platform=MAC}] does not match the current platform LINUX
2017-03-20 14:52:47.307:INFO:osjs.Server:main: jetty-9.2.20.v20161216
2017-03-20 14:52:47.382:INFO:osjsh.ContextHandler:main: Started o.s.j.s.ServletContextHandler@5e3a8624{/,null,AVAILABLE}
2017-03-20 14:52:47.405:INFO:osjs.AbstractConnector:main: Started ServerConnector@73ad2d6{HTTP/1.1,[http/1.1]}{0.0.0.0:4444}
2017-03-20 14:52:47.408:INFO:osjs.Server:main: Started @610ms
14:52:47.408 INFO - Selenium Server is up and running
```

这个时候就应该是一起启动服务了. 再看一个终端连接的窗口. 查看一下服务
```
[root@CentOS7 test]# ps aux | grep java
root     10176  0.7  4.4 2716864 44952 pts/0   Sl+  14:52   0:00 java -jar selenium-server-standalone-3.3.1.jar
root     10213  0.0  0.0 112664   972 pts/1    S+   14:55   0:00 grep --color=auto java
[root@CentOS7 test]#
```
可以看到, 已经启动了.

如果想关闭的话, 可以在启动的终端中安`CTRL+c`,或者
是`kill -9 10176`

**说明:**

`Selenium Server`是指使用jdk来运行这个`selenium-server-standalone-3.3.1.jar`这个jar文件.

它是一个http的服务, 默认在端口号4444侦听.它从客户端接收到请求,来驱动浏览器,做打开网页、提交表单,各种页面验证等事情.

**-port:** 指定Selenium Server的侦听的端口号。如果没有指定port，使用4444。

**-profilesLocation** （仅Firefox）指定Firefox的profile文件位置。什么？不晓得Firefox的profile是干啥用？简单的说，就是将你浏览网站的cookie、历史记录等记录到一个文件夹下面（点击这里查看详细）。 为什么需要这个呢？在默认情况下，Selenium Server会使用一个空的profile文件夹的，也就是它启动的Firefox是一个“干净 ”的浏览器。有时候，这并不是你所想要的，例如，在某些网站，做过的一些设置（如关掉页面上恼人的浮层），如果是“干净”的浏览器，那你的设置就没法生效 的。

**-browserSessionReuse** 可以节省tests运行时间的。在每个test运行的时候，不用重新再次启动Firefox，复用旧的Firefox。

**-userExtensions** firefox的用户扩展。将一段js代码load到selenium里面去，非常有用。对于一些已经实现了使用JS来验证的工具，无缝的集成到 Selenium里面，这意味着原本单个页面的手工执行JS工具，可以通过Selenium自动化来执行这些工具。

更多的选项，可以来查看帮助 `java -jar selenium-server-standalone-3.3.1.jar –help`

#phantomjs

下载phantomjs, 放到项目根目录里.

新建test.php脚本. 内容如下:

```
<?php
namespace Facebook\WebDriver;
use Facebook\WebDriver\Remote\DesiredCapabilities;
use Facebook\WebDriver\Remote\RemoteWebDriver;
require_once('vendor/autoload.php');

header("Content-Type: text/html; charset=UTF-8");
// start Firefox with 5 second timeout
$waitSeconds = 15;  //需等待加载的时间，一般加载时间在0-15秒，如果超过15秒，报错。
$host = 'http://localhost:4444/wd/hub'; // this is the default
//这里使用的是chrome浏览器进行测试，需到http://www.seleniumhq.org/download/上下载对应的浏览器测试插件
//我这里下载的是win32 Google Chrome Driver 2.25版：https://chromedriver.storage.googleapis.com/index.html?path=2.25/

//$capabilities = DesiredCapabilities::chrome();
$capabilities = DesiredCapabilities::phantomjs(); // 这里用的是phantomjs

$driver = RemoteWebDriver::create($host, $capabilities, 5000);
// navigate to 'http://docs.seleniumhq.org/'
$driver->get('https://www.baidu.com/');

echo iconv("UTF-8","GB2312",'标题1')."：" . $driver->getTitle() . "\n";    //cmd.exe中文乱码，所以需转码

$driver->findElement(WebDriverBy::id('kw'))->sendKeys('wwe')->submit();

// 等待新的页面加载完成....
$driver->wait($waitSeconds)->until(
    WebDriverExpectedCondition::visibilityOfElementLocated(
        WebDriverBy::partialLinkText('100shuai')
    )
);
$driver->findElement(WebDriverBy::partialLinkText('100shuai'))->sendKeys('xxx')->click();    //一般点击链接的时候，担心因为失去焦点而抛异常，则可以先调用一下sendKeys，再click


switchToEndWindow($driver); //切换至最后一个window

// 等待加载....
$driver->wait($waitSeconds)->until(
    WebDriverExpectedCondition::visibilityOfElementLocated(
        WebDriverBy::partialLinkText('SmackDown收视率创历史新低')
    )
);
echo iconv("UTF-8","GB2312",'标题2')."：" . $driver->getTitle() . "\n";    //cmd.exe中文乱码，所以需转码

$driver->findElement(WebDriverBy::partialLinkText('SmackDown收视率创历史新低'))->click();

switchToEndWindow($driver); //切换至最后一个window


// 等待加载....
$driver->wait($waitSeconds)->until(
    WebDriverExpectedCondition::titleContains('SmackDown收视率创历史新低')
);
echo iconv("UTF-8","GB2312",'标题3')."：" . $driver->getTitle() . "\n";    //cmd.exe中文乱码，所以需转码


//关闭浏览器
$driver->quit();

//切换至最后一个window
//因为有些网站的链接点击过去带有target="_blank"属性，就新开了一个TAB，而selenium还是定位到老的TAB上，如果要实时定位到新的TAB，则需要调用此方法，切换到最后一个window
function switchToEndWindow($driver){

    $arr = $driver->getWindowHandles();
    foreach ($arr as $k=>$v){
        if($k == (count($arr)-1)){
            $driver->switchTo()->window($v);
        }
    }
}

?>
```

运行此脚本

```
[root@CentOS7 test]# php test.php
����1：百度一下，你就知道
����2：WWE美摔100分 - WWE美国职业摔角在线观看
����3：SmackDown收视率创历史新低 - WWE百分摔角
[root@CentOS7 test]#
```

ok, 成功了. 证明可以用了.


