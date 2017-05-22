---
layout: post
title: 不刷新改变URL:pushstate
categories: 网站建设
description: 不刷新改变URL:pushstate
keywords: 改变, ajax, url
---

有过在亚马逊上的购物经验的人就会发现，网页通过ajax加载，同时又能完

成URL的改变而没有网页跳转刷新的迹象，就像是改变了网页的url.

# 新的解决方案: pushState

然而HTML5的新接口pushState / replaceState就可以比较完美的解决

问题，它避免了改变hash的问题，避免了用户不理解URL的形式感到疑惑，同

时还有onpopstate提供监听，良好响应后退前进。而且它不需要这个URL真

实存在。

## HTML5 的 pushState+Ajax

HTML5提供history接口，把URL以state的形式添加或者替换到浏览器中，

其实现函数正是 pushState 和 replaceState。

## pushState 例子

pushState() 的基本参数是：

`window.history.pushState(state, title, url);`

其中state和title都可以为空，但是推荐不为空，应当创建state来配合popstate监听。

例如，我们通过pushState现改变URL而不刷新页面。

```javascript
var state = ( {

url: ~href, title: ~title, ~additionalKEY: ~additionalVALUE

} );

window.history.pushState(state, ~title, ~href);

```

# demo演示

<button class="rdbtn" background-color:"red" style="width: 100px;" onclick="history.pushState( null, null, '/test-string');alert('看看地址栏是不是改变了URL')">点我试试</button>

## replaceState 同理

`window.history.replaceState( state, ~title, ~href);`

## pushState、replaceState 的区别

pushState()可以创建历史，可以配合popstate事件，而replaceState()则是替换掉当前的URL，不会产生历史。

# 限制因素

只能用同域的URL替换，例如你不能用http://baidu.com去替

换http://google.com。而且state对象不存储不可序列化的对象如

DOM。
