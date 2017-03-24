---
layout: post
title: ThinkPHP的路由
categories: PHP
description: ThinkPHP的路由
keywords: PHP, ThinkPHP, 路由
---

之前分析了ThinkPHP在执行指定的方法之前的步骤. 
下面来看看, 通过路由规则后, 实例化了指定的类, 然后执行指定的方法. 是怎么实现的.


先看看代码段:

---
**code 1**
`/Users/qiuyu/www/thinkphp_3.2.3_full/Application/Home/Controller/IndexController.class.php`
```php
<?php
namespace Home\Controller;
use Think\Controller;

class IndexController extends Controller
{
    public function index()
	{
		$lesson_api = D('lesson');
		$res = $lesson_api->getLast();
dump($res);
	}
}
```
---
首先是继承了Controller类. `Controller.class.php`
这个类中, 有个构造方法.

**code 2**
`/Users/qiuyu/www/thinkphp_3.2.3_full/ThinkPHP/Library/Think/Controller.class.php`

```php
   /**
     * 架构函数 取得模板对象实例
     * @access public
     */
    public function __construct() {
        Hook::listen('action_begin',$this->config);
        //实例化视图类
        $this->view     = Think::instance('Think\View');
        //控制器初始化
        if(method_exists($this,'_initialize'))
            $this->_initialize();
    }
```
作用有3点:
1. 执行方法开始行为
2. 实例化视图类
3. 如果`Controller.class.php`的子类中有`_initialize()`方法, 就执行这个方法. 相当于子类中的构造方法. 

再来看一下`code 1`中的`D()`方法. 

**code 3**
`/Users/qiuyu/www/thinkphp_3.2.3_full/ThinkPHP/Common/functions.php`
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
可以看到. 如果不传参数, 就是实例化`Think\Model`类.
* 第一个参数, 就是要调用的模型层中的类名.
* 第二个参数, 是在模型层分为多层的情况下使用, 指定是哪个模型层. 如果为空, 则按照配置文件, 值就是默认的模型层名称.
* 根据代码`if(isset($_model[$name.$layer]))`可以看出, 是单例模式. 如果存在这个实例, 就直接返回, 防止多次实例化.
* 最后, 根据参数, 实例化对应的类. 如果类不存在, 就实例化默认的`Think\Model`, 同时也把参数1在实例化的时候传过去了. 

简单来说,`D('lesson')`的作用, 与下面的代码是等效的.

**code 4**
```php
<?php
namespace Home\Controller;
use Think\Controller;

class IndexController extends Controller
{
    public function index()
	{
		// $lesson_api = D('lesson');
		$lesson_api = new \Home\Model\LessonModel('lesson'); // 与上面的代码是等效的.
		$res = $lesson_api->getLast();
dump($res);
	}
}
```
既然是实例化的`\Home\Model\LessonModel`, 那么就看看这个类. 

**code 5**
`/Users/qiuyu/www/thinkphp_3.2.3_full/Application/Home/Model/LessonModel.class.php`
```php
<?php
namespace Home\Model;
use Think\Model;

class LessonModel extends Model
{
	public function getLast()
	{
		$res = $this->field('lesson_id, lesson_name')->where('status = 1')->order('lesson_id desc')->limit(10)->select();
		return $res;
	}
}
```

这个Mode类比较复杂. 代码就近2000行, 根据流程一步一步的看下去吧.
首先先看这个类中的构造方法:

**code 6**
`/Users/qiuyu/www/thinkphp_3.2.3_full/ThinkPHP/Library/Think/Model.class.php`
```php
    /**
     * 架构函数
     * 取得DB类的实例对象 字段检查
     * @access public
     * @param string $name 模型名称
     * @param string $tablePrefix 表前缀
     * @param mixed $connection 数据库连接信息
     */
    public function __construct($name='',$tablePrefix='',$connection='') {
        // 模型初始化
        $this->_initialize();
        // 获取模型名称
        if(!empty($name)) {
            if(strpos($name,'.')) { // 支持 数据库名.模型名的 定义
                list($this->dbName,$this->name) = explode('.',$name);
            }else{
                $this->name   =  $name;
            }
        }elseif(empty($this->name)){
            $this->name =   $this->getModelName();
        }
        // 设置表前缀
        if(is_null($tablePrefix)) {// 前缀为Null表示没有前缀
            $this->tablePrefix = '';
        }elseif('' != $tablePrefix) {
            $this->tablePrefix = $tablePrefix;
        }elseif(!isset($this->tablePrefix)){
            $this->tablePrefix = C('DB_PREFIX');
        }
        // 数据库初始化操作
        // 获取数据库操作对象
        // 当前模型有独立的数据库连接信息
        $this->db(0,empty($this->connection)?$connection:$this->connection,true);
    }
```
分析一下这个构造方法的作用:

1. 模型初始化, 作用与Contrller中的一样, 如果Model类的子类中, 有`_initialize()`方法, 则会执行. 为什么要这么写, 可以看看我的另一篇文章, 链接: [ThinkPHP中的__construct()与_initilize()的区别](http://www.qiuyuhome.com/?p=99)
2. 获取模型名称
3. 设置表前缀
4. 数据库初始化操作(重点)
	
最有一句代码, 又执行了这个类中的`db()`方法. 传了3个参数, 等效于`$this->db(0, '', true)`


`db()`方法的代码如下. 

**code 7**
`/Users/qiuyu/www/thinkphp_3.2.3_full/ThinkPHP/Library/Think/Model.class.php`
```php
    /**
     * 切换当前的数据库连接
     * @access public
     * @param integer $linkNum  连接序号
     * @param mixed $config  数据库连接信息
     * @param boolean $force 强制重新连接
     * @return Model
     */
    public function db($linkNum='',$config='',$force=false) {
        if('' === $linkNum && $this->db) {
            return $this->db;
        }

        if(!isset($this->_db[$linkNum]) || $force ) {
            // 创建一个新的实例
            if(!empty($config) && is_string($config) && false === strpos($config,'/')) { // 支持读取配置参数
                $config  =  C($config);
            }
            $this->_db[$linkNum]            =    Db::getInstance($config);
        }elseif(NULL === $config){
            $this->_db[$linkNum]->close(); // 关闭数据库连接
            unset($this->_db[$linkNum]);
            return ;
        }

        // 切换数据库连接
        $this->db   =    $this->_db[$linkNum];
        $this->_after_db();
        // 字段检测
        if(!empty($this->name) && $this->autoCheckFields)    $this->_checkTableInfo();
        return $this;
    }
```
分析:
根据参数, 会执行`$this->_db[$linkNum]            =    Db::getInstance($config);`. 其中$config的值是`""`, 空字符串. 
是db类中的一个静态方法. 这个db类是ThinkPHP的数据库中间层实现类. 只有4个方法.
下面是db类中的`getInstance()`方法.

**code 8**
`/Users/qiuyu/www/thinkphp_3.2.3_full/ThinkPHP/Library/Think/Db.class.php`
```php
    /**
     * 取得数据库类实例
     * @static
     * @access public
     * @param mixed $config 连接配置
     * @return Object 返回数据库驱动类
     */
    static public function getInstance($config=array()) {
        $md5    =   md5(serialize($config));
        if(!isset(self::$instance[$md5])) {
            // 解析连接参数 支持数组和字符串
            $options    =   self::parseConfig($config);
            // 兼容mysqli
            if('mysqli' == $options['type']) $options['type']   =   'mysql';
            // 如果采用lite方式 仅支持原生SQL 包括query和execute方法
            $class  =   !empty($options['lite'])?  'Think\Db\Lite' :   'Think\\Db\\Driver\\'.ucwords(strtolower($options['type']));
            if(class_exists($class)){
                self::$instance[$md5]   =   new $class($options);
            }else{
                // 类没有定义
                E(L('_NO_DB_DRIVER_').': ' . $class);
            }
        }
        self::$_instance    =   self::$instance[$md5];
        return self::$_instance;
    }
```
分析:
这个方法也是一个单例模式, 如果没有实力, 就运行if里面的代码. 其中的代码`$options =   self::parseConfig($config)`,  又调用了`parseconfig()`方法  .参数是空字符串.
下面是这个方法的代码.

**code 9**
```php
    /**
     * 数据库连接参数解析
     * @static
     * @access private
     * @param mixed $config
     * @return array
     */
    static private function parseConfig($config){
        if(!empty($config)){
            if(is_string($config)) {
                return self::parseDsn($config);
            }
            $config =   array_change_key_case($config);
            $config = array (
                'type'          =>  $config['db_type'],
                'username'      =>  $config['db_user'],
                'password'      =>  $config['db_pwd'],
                'hostname'      =>  $config['db_host'],
                'hostport'      =>  $config['db_port'],
                'database'      =>  $config['db_name'],
                'dsn'           =>  isset($config['db_dsn'])?$config['db_dsn']:null,
                'params'        =>  isset($config['db_params'])?$config['db_params']:null,
                'charset'       =>  isset($config['db_charset'])?$config['db_charset']:'utf8',
                'deploy'        =>  isset($config['db_deploy_type'])?$config['db_deploy_type']:0,
                'rw_separate'   =>  isset($config['db_rw_separate'])?$config['db_rw_separate']:false,
                'master_num'    =>  isset($config['db_master_num'])?$config['db_master_num']:1,
                'slave_no'      =>  isset($config['db_slave_no'])?$config['db_slave_no']:'',
                'debug'         =>  isset($config['db_debug'])?$config['db_debug']:APP_DEBUG,
                'lite'          =>  isset($config['db_lite'])?$config['db_lite']:false,
            );
        }else {
            $config = array (
                'type'          =>  C('DB_TYPE'),
                'username'      =>  C('DB_USER'),
                'password'      =>  C('DB_PWD'),
                'hostname'      =>  C('DB_HOST'),
                'hostport'      =>  C('DB_PORT'),
                'database'      =>  C('DB_NAME'),
                'dsn'           =>  C('DB_DSN'),
                'params'        =>  C('DB_PARAMS'),
                'charset'       =>  C('DB_CHARSET'),
                'deploy'        =>  C('DB_DEPLOY_TYPE'),
                'rw_separate'   =>  C('DB_RW_SEPARATE'),
                'master_num'    =>  C('DB_MASTER_NUM'),
                'slave_no'      =>  C('DB_SLAVE_NO'),
                'debug'         =>  C('DB_DEBUG',null,APP_DEBUG),
                'lite'          =>  C('DB_LITE'),
            );
        }
        return $config;
    }
```

参数为空, 可以看到. 会获取数据库的配置信息. 然后返回. 继续执行`code 8`中的代码. 
重要的代码:
` $class  =   !empty($options['lite'])?  'Think\Db\Lite' :   'Think\\Db\\Driver\\'.ucwords(strtolower($options['type']));`
`$class`的值为`Think\Db\Driver\Mysql`. 

然后会实例化`$class`, 并且把获取到的数据库配置信息传过去. 
`Think\Db\Driver\Mysql`这个类没有构造方法, 但继承了`Driver`,

**code 10**
`/Users/qiuyu/www/thinkphp_3.2.3_full/ThinkPHP/Library/Think/Db/Driver/Mysql.class.php`
```php
namespace Think\Db\Driver;
use Think\Db\Driver;

/**
 * mysql数据库驱动 
 */
class Mysql extends Driver{
	...
	...
	...
}
```
那么再看看它的父类.

**code 11**
`/Users/qiuyu/www/thinkphp_3.2.3_full/ThinkPHP/Library/Think/Db/Driver.class.php`
```php
namespace Think\Db;
use Think\Config;
use Think\Debug;
use Think\Log;
use PDO;

abstract class Driver {
	...
	...
	...
}
```
这个类中定义了数据的操作方法. 如最常用的`select(), find(), where()`等等. 
可以看到. 父类是个抽象类. 但是没有抽象方法. 也就是说. 这个类只能被继承. 不能实例化. 
仔细想想, ThinkPHP的核心是CBD, 也就是:***核心, 行为, 驱动***.  这个应该就是驱动的实现方式. 说的挺高大上, 直接说是抽象类, 大家不就都明白了么. 靠

再来看看这个类中的方法, 有构造方法. 接受的参数是数据库的配置信息.如下:

**code 12**
`/Users/qiuyu/www/thinkphp_3.2.3_full/ThinkPHP/Library/Think/Db/Driver.class.php`
```php
    /**
     * 架构函数 读取数据库配置信息
     * @access public
     * @param array $config 数据库配置数组
     */
    public function __construct($config=''){
        if(!empty($config)) {
            $this->config   =   array_merge($this->config,$config);
            if(is_array($this->config['params'])){
                $this->options  =   $this->config['params'] + $this->options;
            }
        }
    }
```
这个构造方法的作用, 就是把数据库的配置信息, 加载入类中的属性中 `$this->options`. 方便以后的调用. 

还有一个构造方法

**code 13**
`/Users/qiuyu/www/thinkphp_3.2.3_full/ThinkPHP/Library/Think/Db/Driver.class.php`
```php
   /**
     * 析构方法
     * @access public
     */
    public function __destruct() {
        // 释放查询
        if ($this->PDOStatement){
            $this->free();
        }
        // 关闭连接
        $this->close();
    }
```

ok. 按照流程, 代码现在没有能够执行的了. 需要返回. 也就是说, `code 8`中的代码`self::$instance[$md5]   =   new $class($options);`执行完毕了. 继续执行`code 8`中的下面的代码. 

再返回到`code 7`中的代码, 这个时候, 代码`$this->_db[$linkNum]            =    Db::getInstance($config);`已经执行完毕了. 
输出`$this->_db[$linkNum]`, 结果是:

```php
object(Think\Db\Driver\Mysql)#7 (17) {
  ["PDOStatement":protected] => NULL
  ["model":protected] => string(7) "_think_"
  ["queryStr":protected] => string(0) ""
  ["modelSql":protected] => array(0) {
  }
  ["lastInsID":protected] => NULL
  ["numRows":protected] => int(0)
  ["transTimes":protected] => int(0)
  ["error":protected] => string(0) ""
  ["linkID":protected] => array(0) {
  }
  ["_linkID":protected] => NULL
  ["config":protected] => array(17) {
    ["type"] => string(5) "mysql"
    ["hostname"] => string(13) "192.168.1.186"
    ["database"] => string(12) "zhulong_edu3"
    ["username"] => string(3) "sns"
    ["password"] => string(6) "123456"
    ["hostport"] => string(4) "3306"
    ["dsn"] => NULL
    ["params"] => array(0) {
    }
    ["charset"] => string(4) "utf8"
    ["prefix"] => string(0) ""
    ["debug"] => bool(true)
    ["deploy"] => int(0)
    ["rw_separate"] => bool(false)
    ["master_num"] => int(1)
    ["slave_no"] => string(0) ""
    ["db_like_fields"] => string(0) ""
    ["lite"] => NULL
  }
  ["exp":protected] => array(14) {
    ["eq"] => string(1) "="
    ["neq"] => string(2) "<>"
    ["gt"] => string(1) ">"
    ["egt"] => string(2) ">="
    ["lt"] => string(1) "<"
    ["elt"] => string(2) "<="
    ["notlike"] => string(8) "NOT LIKE"
    ["like"] => string(4) "LIKE"
    ["in"] => string(2) "IN"
    ["notin"] => string(6) "NOT IN"
    ["not in"] => string(6) "NOT IN"
    ["between"] => string(7) "BETWEEN"
    ["not between"] => string(11) "NOT BETWEEN"
    ["notbetween"] => string(11) "NOT BETWEEN"
  }
  ["selectSql":protected] => string(109) "SELECT%DISTINCT% %FIELD% FROM %TABLE%%FORCE%%JOIN%%WHERE%%GROUP%%HAVING%%ORDER%%LIMIT% %UNION%%LOCK%%COMMENT%"
  ["queryTimes":protected] => int(0)
  ["executeTimes":protected] => int(0)
  ["options":protected] => array(4) {
    [8] => int(2)
    [3] => int(2)
    [11] => int(0)
    [17] => bool(false)
  }
  ["bind":protected] => array(0) {
  }
}
```
这些信息. 都是`/Users/qiuyu/www/thinkphp_3.2.3_full/ThinkPHP/Library/Think/Db/Driver.class.php`这个类中的属性. 还有就是数据库的配置信息. 

然后, Model的构造方法, 也就是`code 7`中返回了这个实例化后的实例.
这样, 我们在Model层中, 就已经获得了数据库的实例, 只需要像`code 5`那样, 继承了Model之后, 就可以直接写类似于`$this->query()`, `$this->find()`的语句. 进行数据库操作. 





