---
layout: post
title: ThinkPHP中对Behavior的理解
categories: PHP
description: ThinkPHP中对Behavior的理解
keywords: PHP, ThinkPHP
---

# ThinkPHP中对Behavior的理解

官方手册必须要先看几遍的. [CBD模式](http://www.kancloud.cn/manual/thinkphp/1699)

文档中说明了基本的概念:

**Behavior（行为）**

行为（Behavior）是ThinkPHP扩展机制中比较关键的一项扩展，行为既可以独立调用，也可以绑定到某个标签（位）中进行侦听。这里的行为指的是一个比较抽象的概念，你可以想象成在应用执行过程中的一个动作或者处理，在框架的执行流程中，各个位置都可以有行为产生，例如路由检测是一个行为，静态缓存是一个行为，用户权限检测也是行为，大到业务逻辑，小到浏览器检测、多语言检测等等都可以当做是一个行为，甚至说你希望给你的网站用户的第一次访问弹出Hello，world！这些都可以看成是一种行为，行为的存在让你无需改动框架和应用，而在外围通过扩展或者配置来改变或者增加一些功能。

而不同的行为之间也具有位置共同性，比如，有些行为的作用位置都是在应用执行前，有些行为都是在模板输出之后，我们把这些行为发生作用的位置称之为**标签（位）**，也可以称之为**钩子**，当应用程序运行到这个标签的时候，就会被拦截下来，统一执行相关的行为，类似于AOP编程中的“切面”的概念，给某一个标签绑定相关行为就成了一种类AOP编程的思想。


**需要解决的问题:**
1. 行为和标签位有什么用?
2. 这么用有什么好处? 为什么要这么用?
3. 怎么用?
4. 原理是什么? 怎么实现的?


===
在`/ThinkPHP/Library/Think/Think.class.php`的`start()`中, 加载了行为的配置文件.

```php
// 加载模式行为定义
if(isset($mode['tags'])) {
    Hook::import(is_array($mode['tags'])?$mode['tags']:include $mode['tags']);
}
// 加载应用行为定义
if(is_file(CONF_PATH.'tags.php'))
    // 允许应用增加开发模式配置定义
    Hook::import((include CONF_PATH.'tags.php'), true, 1);   

```

加载了什么呢? 先来看一下系统自带的配置文件:
![-w825](/images/posts/14827486260286.jpg)


这里用到了`Hook::import()`, 他的作用就是把配置文件都合并到一个私有的属性里. 是个数组. 
文件是:`/ThinkPHP/Library/Think/Hook.class.php`, 

![-w491](/images/posts/14827481478003.jpg)
I

```php
    /**
     * 批量导入插件
     * @param array $data 插件信息
     * @param boolean $recursive 是否递归合并
     * @return void
     */
    static public function import($data,$recursive=true) {
        if(!$recursive){ // 覆盖导入
            self::$tags   =   array_merge(self::$tags,$data);
        }else{ // 合并导入
            foreach ($data as $tag=>$val){
                if(!isset(self::$tags[$tag]))
                    self::$tags[$tag]   =   array();            
                if(!empty($val['_overlay'])){
                    // 可以针对某个标签指定覆盖模式
                    unset($val['_overlay']);
                    self::$tags[$tag]   =   $val;
                }else{
                    // 合并模式
                    self::$tags[$tag]   =   array_merge(self::$tags[$tag],$val);
                }
            }            
        }
    }
```

到这里, 现在`/ThinkPHP/Library/Think/Hook.class.php`的私有属性`$tags`里面已经存在了配置文件中设置的tags. 也就是第一个图中的数据.


怎么使用呢. 源码中就有例子:
`/ThinkPHP/Library/Think/App.class.php`中使用了`Hook::listen()`.

```
    /**
     * 运行应用实例 入口文件使用的快捷方法
     * @access public
     * @return void
     */
    static public function run() {
        // 应用初始化标签
        Hook::listen('app_init');
        App::init();
        // 应用开始标签
        Hook::listen('app_begin');
        // Session初始化
        if(!IS_CLI){
            session(C('SESSION_OPTIONS'));
        }
        // 记录应用初始化时间
        G('initTime');
        App::exec();
        // 应用结束标签
        Hook::listen('app_end');
        return ;
    }
```

`Hook::listen()`是干什么的呢? 来看一下. 第一次使用`Hook:listen()`出入了参数:`app_init`. 我们来跟踪它.
`/ThinkPHP/Library/Think/Hook.class.php`

```
    /**
     * 监听标签的插件
     * @param string $tag 标签名称
     * @param mixed $params 传入参数
     * @return void
     */
    static public function listen($tag, &$params=NULL) {
        if(isset(self::$tags[$tag])) {
            if(APP_DEBUG) {
                G($tag.'Start');
                trace('[ '.$tag.' ] --START--','','INFO');
            }
            foreach (self::$tags[$tag] as $name) {
                APP_DEBUG && G($name.'_start');
                $result =   self::exec($name, $tag,$params);
                if(APP_DEBUG){
                    G($name.'_end');
                    trace('Run '.$name.' [ RunTime:'.G($name.'_start',$name.'_end',6).'s ]','','INFO');
                }
                if(false === $result) {
                    // 如果返回false 则中断插件执行
                    return ;
                }
            }
            if(APP_DEBUG) { // 记录行为的执行日志
                trace('[ '.$tag.' ] --END-- [ RunTime:'.G($tag.'Start',$tag.'End',6).'s ]','','INFO');
            }
        }
        return;
    }
```

可以发现. 这里会判断`tags`是否包含了传过来的数据. 因为第一个图的配置文件都已经合并到了tags里, 所以, 这里是包含的. 那么就会执行`self::exec()`.

```php
    /**
     * 执行某个插件
     * @param string $name 插件名称
     * @param string $tag 方法名（标签名）     
     * @param Mixed $params 传入的参数
     * @return void
     */
    static public function exec($name, $tag,&$params=NULL) {
        if('Behavior' == substr($name,-8) ){
            // 行为扩展必须用run入口方法
            $tag    =   'run';
        }
        $addon   = new $name();
        return $addon->$tag($params);
    }
```

这里会实例化`Behavior\BuildLiteBehavior`, 然后运行这个类中的run方法.
这样, 就触发了插件.



===
问题4已经给出答案了. 那么前3个问题就自己思考吧.

===














