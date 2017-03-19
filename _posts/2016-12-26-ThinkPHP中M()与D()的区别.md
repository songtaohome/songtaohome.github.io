---
layout: post
title: ThinkPHP中M()与D()的区别
categories: PHP
description: ThinkPHP中M()与D()的区别
keywords: PHP, ThinkPHP
---

# ThinkPHP中M()与D()的区别

### 需要解决的问题:
什么时候用M(), 什么时候用D().

### 解决方案的来源: 
自己查看源代码

### 分析:

先看一下源代码

```php
/**
 * 实例化一个没有模型文件的Model
 * @param string $name Model名称 支持指定基础模型 例如 MongoModel:User
 * @param string $tablePrefix 表前缀
 * @param mixed $connection 数据库连接信息
 * @return Think\Model
 */
function M($name='', $tablePrefix='',$connection='') {
    static $_model  = array();
    if(strpos($name,':')) {
        list($class,$name)    =  explode(':',$name);
    }else{
        $class      =   'Think\\Model';
    }
    $guid           =   (is_array($connection)?implode('',$connection):$connection).$tablePrefix . $name . '_' . $class;
	if (!isset($_model[$guid]))
        $_model[$guid] = new $class($name,$tablePrefix,$connection);
    return $_model[$guid];
}
```

```php
/**
 * 实例化模型类 格式 [资源://][模块/]模型
 * @param string $name 资源地址
 * @param string $layer 模型层名称
 * @return Think\Model
 */
function D($name='',$layer='') {
    if(empty($name)) return new Think\Model;
    static $_model  =   array();
    $layer          =   $layer? : C('DEFAULT_M_LAYER');
    if(isset($_model[$name.$layer]))
        return $_model[$name.$layer];
    $class          =   parse_res_name($name,$layer);
    if(class_exists($class)) {
        $model      =   new $class(basename($name));
    }elseif(false === strpos($name,'/')){
        // 自动加载公共模块下面的模型
        if(!C('APP_USE_NAMESPACE')){
            import('Common/'.$layer.'/'.$class);
        }else{
            $class      =   '\\Common\\'.$layer.'\\'.$name.$layer;
        }
        $model      =   class_exists($class)? new $class($name) : new Think\Model($name);
    }else {
        Think\Log::record('D方法实例化没找到模型类'.$class,Think\Log::NOTICE);
        $model      =   new Think\Model(basename($name));
    }
    $_model[$name.$layer]  =  $model;
    return $model;
}
```
```
	/**
	 * 测试方法
	 */
	public function demo(){
		// $obj = D('user');
		$obj = M('user');
		$res = $obj->select();
		dump($res);
		die;
	}
```

![-w505](/images/posts/14827371831794.jpg)


输出结果:
![-w532](/images/posts/14827372326314.jpg)

配置文件: 其他的配置用的都是默认的.
![-w823](/images/posts/14827372873967.jpg)








