---
layout: post
title: Python Selenium PhantomJS
categories: Python
description: Python Selenium PhantomJS
keywords: Python, Selenium, PhantomJS
---

# Python Selenium PhantomJS

```
from selenium import webdriver
 
browser = webdriver.Chrome()
    browser.get('http://www.baidu.com/')
```

报错. 

```
➜  test python demo.py
Traceback (most recent call last):
  File "demo.py", line 3, in <module>
    browser = webdriver.Chrome()
  File "/Library/Python/2.7/site-packages/selenium/webdriver/chrome/webdriver.py", line 62, in __init__
    self.service.start()
  File "/Library/Python/2.7/site-packages/selenium/webdriver/common/service.py", line 81, in start
    os.path.basename(self.path), self.start_error_message)
selenium.common.exceptions.WebDriverException: Message: 'chromedriver' executable needs to be in PATH. Please see https://sites.google.com/a/chromium.org/chromedriver/home

➜  test
```

解决方案:

下载谷歌浏览器驱动. 
https://sites.google.com/a/chromium.org/chromedriver/downloads

解压后, 这个文件的目录加入到环境变量中就可以了. 
(注意, 解压后得到的是一个文件. 不能把这个文件加入到环境变量中. )









