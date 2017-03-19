---
layout: post
title: smarty中的foreach遍历和时间转换
categories: HTML
description: smarty中的foreach遍历和时间转换
keywords: smarty
---

## 在smarty中使用foreach

```html
<html>
    <head>
        <meta http-equiv="content-type" content="text/html; charset=utf-8">
        <title>demo</title>
    </head>
    <body>
     	<center><h1>demo</h1></center>
     	<hr/>
			<div>
				<{foreach from = $lists item = list name = name}>
				<p>
					<!-- total是这个数组的总数, 相当于php中的count($lists) -->
					<{$smarty.foreach.name.total}> | 

					<!-- index是这个数组的数字索引, 从0开始 -->
					<{$smarty.foreach.name.index}> | 

					<!-- interation也是数字索引, 不同的是, 他是从1开始 -->
					<{$smarty.foreach.name.iteration}> | 
					
					<!-- first是第一个元素  lasts是最后一个元素-->
					<{$smarty.foreach.name.first}><{$smarty.foreach.name.last}>
				</p>
				<{/foreach}>
			</div>
    </body>
    <script type="text/javascript" charset="utf-8">
		
	</script>
</html>
```

## 格式化时间

在Smarty 中获取当前日期时间和格式化日期时间与PHP中有些不同的地方，这里就为您详细介绍：

首先是获取当前的日期时间：
在PHP中我们会使用date函数来获取当前的时间，实例代码如下：

```
date("Y-m-dH:i:s");   //该结果会显示为：2010-07-27 21:19:36 的模式
```

但是在Smarty 模板中我们就不能使用date 了，而是应该使用 now 来获取当前的时间，实例代码如下：

```
{$smarty.now}      //该结果会显示为：1280236776的时间戳模式
```

然而我们还可以将这个时间戳格式化，实例代码如下：

```
{$smarty.now|date_format:'%Y-%m-%d %H:%M:%S'}   //该结果会显示为 2010-07-27 21:19:36 的时间模式
```

需要说明的是 `Smarty` 中的这个`date_format` 时间格式化函数和PHP中的 `strftime()`函数基本上相同，您可以去查看PHP中的 `strftime()` 函数中的`format` 识别转换标记。

其中 `%Y` 是代表十进制年份，`%m`是代表十进制月份，`%d` 是代表十进制天数，`%H` 是代表十进制小时数，`%M`是代表十进制的分数，`%S`是代表十进制的秒数（这里的S是大写的哦）。  



