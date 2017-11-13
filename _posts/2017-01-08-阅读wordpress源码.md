---
layout: post
title: 阅读wordpress源码
categories: PHP
description: 阅读wordpress源码
keywords: php, wordpress
---

wordpress最为使用最广泛的开源博客, 在国内也有很大的用户量, 多了解一些肯定是没错的. 听说wordpress是面向过程的. 具体的还得看源码啊.

## 声明:
1. **wordpress版本:** 4.5.4 中文版
2. **目前对wordpress的了解程度:** 没有阅读过源码.
3. **阅读源码的目的:** 了解加载机制, 能够自己开发主题和插件. 达到随意扩展功能的目的.
4. **预计学习周期:** 3个周末.
5. **在学习阶段, 会全程记录我的学习笔记. 方便以后查看. 同时也是为了分享.**

下面进入正题. 开始阅读WordPress的源码. 

首先, wordpress是单入口框架. 说先进入根目录下的`index.php`文件. 

**code 1**
`/Users/qiuyu/www/blog/index.php`

```php
<?php
/**
 * Front to the WordPress application. This file doesn't do anything, but loads
 * wp-blog-header.php which does and tells WordPress to load the theme.
 *
 * @package WordPress
 */

/**
 * Tells WordPress to load the WordPress theme and output it.
 *
 * @var bool
 */
define('WP_USE_THEMES', true);

/** Loads the WordPress Environment and Template */
require( dirname( __FILE__ ) . '/wp-blog-header.php' );
```

只有2行的代码, 作用是:
1. 定义是否使用主题的常量, 默认是开启
2. 引入了根目录下的`wp-blog-header.php`文件.

**code 2**
`/Users/qiuyu/www/blog/wp-blog-header.php`

```php
<?php
/**
 * Loads the WordPress environment and template.
 *
 * @package WordPress
 */

if ( !isset($wp_did_header) ) {

	$wp_did_header = true;

	// Load the WordPress library.
	require_once( dirname(__FILE__) . '/wp-load.php' );

	// Set up the WordPress query.
	wp();

	// Load the theme template.
	require_once( ABSPATH . WPINC . '/template-loader.php' );

}
```

这个就是一个类似于单例模式的文件, 确保只载入一次. 
作用:
1. 只加载一次. 
2. 引入根目录下的`wp-load.php`文件.
3. 执行`wp()`函数.
4. 引入`/Users/qiuyu/www/blog/wp-includes/template-loader.php`文件.

ok, 继续, 看看`wp-load.php`的文件吧. 

**code 3**
`/Users/qiuyu/www/blog/wp-load.php`

```php
<?php
/**
 * Bootstrap file for setting the ABSPATH constant
 * and loading the wp-config.php file. The wp-config.php
 * file will then load the wp-settings.php file, which
 * will then set up the WordPress environment.
 *
 * If the wp-config.php file is not found then an error
 * will be displayed asking the visitor to set up the
 * wp-config.php file.
 *
 * Will also search for wp-config.php in WordPress' parent
 * directory to allow the WordPress directory to remain
 * untouched.
 *
 * @internal This file must be parsable by PHP4.
 *
 * @package WordPress
 */

/** Define ABSPATH as this file's directory */
define( 'ABSPATH', dirname(__FILE__) . '/' );

error_reporting( E_CORE_ERROR | E_CORE_WARNING | E_COMPILE_ERROR | E_ERROR | E_WARNING | E_PARSE | E_USER_ERROR | E_USER_WARNING | E_RECOVERABLE_ERROR );

/*
 * If wp-config.php exists in the WordPress root, or if it exists in the root and wp-settings.php
 * doesn't, load wp-config.php. The secondary check for wp-settings.php has the added benefit
 * of avoiding cases where the current directory is a nested installation, e.g. / is WordPress(a)
 * and /blog/ is WordPress(b).
 *
 * If neither set of conditions is true, initiate loading the setup process.
 */
if ( file_exists( ABSPATH . 'wp-config.php') ) {

	/** The config file resides in ABSPATH */
	require_once( ABSPATH . 'wp-config.php' );

} elseif ( @file_exists( dirname( ABSPATH ) . '/wp-config.php' ) && ! @file_exists( dirname( ABSPATH ) . '/wp-settings.php' ) ) {

	/** The config file resides one level above ABSPATH but is not part of another install */
	require_once( dirname( ABSPATH ) . '/wp-config.php' );

} else {

	// A config file doesn't exist

	define( 'WPINC', 'wp-includes' );
	require_once( ABSPATH . WPINC . '/load.php' );

	// Standardize $_SERVER variables across setups.
	wp_fix_server_vars();

	require_once( ABSPATH . WPINC . '/functions.php' );

	$path = wp_guess_url() . '/wp-admin/setup-config.php';

	/*
	 * We're going to redirect to setup-config.php. While this shouldn't result
	 * in an infinite loop, that's a silly thing to assume, don't you think? If
	 * we're traveling in circles, our last-ditch effort is "Need more help?"
	 */
	if ( false === strpos( $_SERVER['REQUEST_URI'], 'setup-config' ) ) {
		header( 'Location: ' . $path );
		exit;
	}

	define( 'WP_CONTENT_DIR', ABSPATH . 'wp-content' );
	require_once( ABSPATH . WPINC . '/version.php' );

	wp_check_php_mysql_versions();
	wp_load_translations_early();

	// Die with an error message
	$die  = sprintf(
		/* translators: %s: wp-config.php */
		__( "There doesn't seem to be a %s file. I need this before we can get started." ),
		'<code>wp-config.php</code>'
	) . '</p>';
	$die .= '<p>' . sprintf(
		/* translators: %s: Codex URL */
		__( "Need more help? <a href='%s'>We got it</a>." ),
		__( 'https://codex.wordpress.org/Editing_wp-config.php' )
	) . '</p>';
	$die .= '<p>' . sprintf(
		/* translators: %s: wp-config.php */
		__( "You can create a %s file through a web interface, but this doesn't work for all server setups. The safest way is to manually create the file." ),
		'<code>wp-config.php</code>'
	) . '</p>';
	$die .= '<p><a href="' . $path . '" class="button button-large">' . __( "Create a Configuration File" ) . '</a>';

	wp_die( $die, __( 'WordPress &rsaquo; Error' ) );
}
```

此文件的作用:
1. 定义根目录的常量: ABSPATH.
2. 设置报错等级.
3. 判断根目录下的配置文件`wp-config.php`是否存在, 不存在, 就开始配置引导页面.(我这里已经存在, 就先不看不存在的代码). 如果存在, 就引入这个配置文件.

下面是`wp-config.php`的代码.

**code 4**
`/Users/qiuyu/www/blog/wp-config.php`

```php
<?php
/**
 * WordPress基础配置文件。
 *
 * 这个文件被安装程序用于自动生成wp-config.php配置文件，
 * 您可以不使用网站，您需要手动复制这个文件，
 * 并重命名为“wp-config.php”，然后填入相关信息。
 *
 * 本文件包含以下配置选项：
 *
 * * MySQL设置
 * * 密钥
 * * 数据库表名前缀
 * * ABSPATH
 *
 * @link https://codex.wordpress.org/zh-cn:%E7%BC%96%E8%BE%91_wp-config.php
 *
 * @package WordPress
 */

// ** MySQL 设置 - 具体信息来自您正在使用的主机 ** //
/** WordPress数据库的名称 */
define('DB_NAME', 'wordpress');

/** MySQL数据库用户名 */
define('DB_USER', 'root');

/** MySQL数据库密码 */
define('DB_PASSWORD', '');

/** MySQL主机 */
define('DB_HOST', 'localhost');

/** 创建数据表时默认的文字编码 */
define('DB_CHARSET', 'utf8mb4');

/** 数据库整理类型。如不确定请勿更改 */
define('DB_COLLATE', '');

/**#@+
 * 身份认证密钥与盐。
 *
 * 修改为任意独一无二的字串！
 * 或者直接访问{@link https://api.wordpress.org/secret-key/1.1/salt/
 * WordPress.org密钥生成服务}
 * 任何修改都会导致所有cookies失效，所有用户将必须重新登录。
 *
 * @since 2.6.0
 */
define('AUTH_KEY',         '3EvGc/29}sj<yDq#NhZ^F`T~mVqX zQ8 m;v5}ubI)^t Bv!7:1l`R5x20~,^9&h');
define('SECURE_AUTH_KEY',  'O+Z=N+2.%BYB,,+wS}Sd_Y_eJsGf++Bl6O4P~`JEJlg*Adxf0R@hYh)83PQQEzTl');
define('LOGGED_IN_KEY',    '?k:o~[<<S&itxsU)%|QS{C#/JlvK;jGKF@llVWu`Rem+bY5Be|EhUY1bL0c|I#.m');
define('NONCE_KEY',        'O^!O[oA;|Z8L8n^ci5YvzP%UvF(MPCgCMdGTIJpS5BRNj%z*hx+:.Fx:Zc_$AzO0');
define('AUTH_SALT',        'Pg[4Rvl/aq8JJ)B4w7@Y_qYHFlc{/XkJE>*:%i`1&<Fulnc?-n]| FDS5S9pbnb1');
define('SECURE_AUTH_SALT', '+wn;IF/&-xddNKA><^]5E]f5<0W3561pF8amRPNSi ]sm{zd23tyfi[CwU;U`flB');
define('LOGGED_IN_SALT',   '9I$eXG_|*j)mJJZ:3zzaY;z2s;8{KVI>OLEf:iB.iw,LE$-`%[i0q+: [LVC+^B(');
define('NONCE_SALT',       'Mkm$7Za/mDc7-p3s8aQii-|GIb6s6uT9k>`zibDrLNScATDEu4IYl)<;$aY.:w_W');

/**#@-*/

/**
 * WordPress数据表前缀。
 *
 * 如果您有在同一数据库内安装多个WordPress的需求，请为每个WordPress设置
 * 不同的数据表前缀。前缀名只能为数字、字母加下划线。
 */
$table_prefix  = 'wp_';

/**
 * 开发者专用：WordPress调试模式。
 *
 * 将这个值改为true，WordPress将显示所有用于开发的提示。
 * 强烈建议插件开发者在开发环境中启用WP_DEBUG。
 *
 * 要获取其他能用于调试的信息，请访问Codex。
 *
 * @link https://codex.wordpress.org/Debugging_in_WordPress
 */
define('WP_DEBUG', false);

/**
 * zh_CN本地化设置：启用ICP备案号显示
 *
 * 可在设置→常规中修改。
 * 如需禁用，请移除或注释掉本行。
 */
define('WP_ZH_CN_ICP_NUM', true);

/* 好了！请不要再继续编辑。请保存本文件。使用愉快！ */

/** WordPress目录的绝对路径。 */
if ( !defined('ABSPATH') )
	define('ABSPATH', dirname(__FILE__) . '/');

/** 设置WordPress变量和包含文件。 */
require_once(ABSPATH . 'wp-settings.php');
```

作用:
1. 数据库的配置信息. 包括:
	* 数据库名称.
	* 数据库用户名.
	* 数据库密码.
	* 数据库地址.
	* 创建数据表时默认的文字编码
2. 身份认证密钥与盐, 作用:
	* 任何修改都会导致所有cookies失效，所有用户将必须重新登录
3. WordPress数据表前缀, 作用:
	* 如果您有在同一数据库内安装多个WordPress的需求，请为每个WordPress设置
4. 是否开启开发者模式, 默认不开启.
5. 是否显示备案号信息.
6. 如果没有定义根目录的常量, 这里就定义.
7. 引入了根目录下的`wp-settings.php`文件.

下面来看一下, `wp-settings.php`文件的内容. 因为这个文件较长, 这里就不列出代码了. 

**code 5**
`/Users/qiuyu/www/blog/wp-settings.php`

```php
// 因为这个文件较长, 这里就不列出代码了(偷懒模式开启. 哈哈)
```

这个文件的作用: 
1. 定义根目录下的文件夹`wp-includes`, 为常量`WPINC`.
	* `define( 'WPINC', 'wp-includes' );`
2. 引入了`wp-includes`下的`load.php`文件. 
	* 这个文件里都是一些函数. 
3. 引入了`wp-includes`下的`default-constants.php`文件
	* 这个文件里都是一些函数.
4. 引入了`wp-includes`下的`version.php`文件.
	* 这个文件里是一些版本信息, 如:php, mysql. 
	* 注意: 这里不是你真是的版本信息, 而是wp定义的版本信息. 后面会比较版本的. 
5. 执行`wp_initial_constants()`函数
	* 位于`/Users/qiuyu/www/blog/wp-includes/default-constants.php`
	* 作用: 这个函数就是定义常量信息的. 具体内容如下.
		1. 没有定义wp的内存限制常量`WP_MEMORY_LIMIT`, 就定义.
			* 多站点: `define('WP_MEMORY_LIMIT', '64M')`
			* 单站点: `define('WP_MEMORY_LIMIT', '40M')`
		2. 没有定义最大内存限制常量`WP_MAX_MEMORY_LIMIT`, 就定义. 
			* `define( 'WP_MAX_MEMORY_LIMIT', '256M' )`
		3. 没有设置`$blog_id `, 就赋值为1. 
		4. 读取php配置文件中的php使用内存大小, 如果比自定义的内存限制大小小, 就重赋值.
			* `ini_set( 'memory_limit', WP_MEMORY_LIMIT )`
		5. 定义常量`WP_CONTENT_DIR`
			* `define( 'WP_CONTENT_DIR', ABSPATH . 'wp-content' )`
		6. 如果前面没有定义常量`WP_DEBUG`, 就定义. 
			* `define( 'WP_DEBUG', false )`
			* 作用: WordPress中的这个 WP_DEBUG常量相信大部分开发者都了解，在wp-config.php 文件下通过对定义这个常量即可开启debug 模式, 就是开启调试模式. 
		7. 如果前面没有定义常量`WP_DEBUG_DISPLAY`, 就定义
			* `define( 'WP_DEBUG_DISPLAY', true )`
			* 作用: 默认的话，在debug 模式下，WordPress 会将大部分的错误显示在前端屏幕上（亦有部分可以通过浏览器的查看源代码发现）。如果你不想显示，可以通过下面的变量关闭之：`define( 'WP_DEBUG_DISPLAY', false )`
		8. 如果前面没有定义常量`WP_DEBUG_LOG`, 就定义
			* `define('WP_DEBUG_LOG', false)`
			* 作用: 通过定义这个常量，WordPress 中会输出debug 的错误信息在wp-content 文件夹下以debug.log 保存，这样你就就可以方便快捷地查看所有的错误并进行修改.
		9. 如果前面没有定义常量`WP_CACHE`, 就定义
			* `define('WP_CACHE', false)`
			* 作用: 默认的话，WordPress对于核心的脚本文件或样式文件会进行压缩化的处理，但在实际开发中，你可能因为要寻找脚本冲突问题而希望可以是不要压缩，那么通过定义这个变量就可.
		10. 如果前面没有定义常量`MEDIA_TRASH`, 就定义
			* `define('MEDIA_TRASH', false)`
			* 作用: 定义是否激活媒体的回收站
		11. 如果前面没有定义常量`SHORTINIT`, 就定义
			* `define('SHORTINIT', false)`
			* 作用: 定义之后，将 load 最小化的 WordPress.
		12. 定义常量`WP_FEATURE_BETTER_PASSWORDS`
			* `define( 'WP_FEATURE_BETTER_PASSWORDS', true );`
			* 作用: Constants for features added to WP that should short-circuit their plugin implementations, (没看懂)
		13. 定义年, 月, 周, 天, 小时, 分钟有多少秒的常量. 
			* define( 'MINUTE_IN_SECONDS', 60 );
			* define( 'HOUR_IN_SECONDS',   60 * MINUTE_IN_SECONDS );
			* define( 'DAY_IN_SECONDS',    24 * HOUR_IN_SECONDS   );
			* define( 'WEEK_IN_SECONDS',    7 * DAY_IN_SECONDS    );
			* define( 'MONTH_IN_SECONDS',  30 * DAY_IN_SECONDS    );
			* define( 'YEAR_IN_SECONDS',  365 * DAY_IN_SECONDS    );
		14. 定义TB, GB, MB, KB多少bytes的常量
			* define( 'KB_IN_BYTES', 1024 );
			* define( 'MB_IN_BYTES', 1024 * KB_IN_BYTES );
			* define( 'GB_IN_BYTES', 1024 * MB_IN_BYTES );
			* define( 'TB_IN_BYTES', 1024 * GB_IN_BYTES );
6. 执行`wp_check_php_mysql_versions();`函数.
	* 位于`/Users/qiuyu/www/blog/wp-includes/load.php`
	* 作用: 检测php, mysql版本的函数, 如果版本低, 就报错.
7. 定义php配置文件. 
	* `ini_set( 'magic_quotes_runtime', 0 )`, 外部引入的文件不加反斜线
	* `ini_set( 'magic_quotes_sybase',  0 )`, 注册变量时, 不转义.
8. 定义时区:
	* `date_default_timezone_set( 'UTC' );`
9. 执行`wp_unregister_GLOBALS()`函数
	* 位于`/Users/qiuyu/www/blog/wp-includes/load.php`
	* 作用: 判断php配置文件是否开启了`register_globals`, 是否注册全局变量. `php.ini`里面的`register_globals=on`时, 是非常危险的. 这个函数就是在`register_globals=on`时会报错. 
10. 执行`wp_fix_server_vars()`函数
	* 位于`/Users/qiuyu/www/blog/wp-includes/load.php`
	* 作用: 处理`$_SERVER`, 且定义了全局变量`$PHP_SELF = $_SERVER['PHP_SELF']`.
11. 执行`wp_favicon_request()`函数
	* 位于`/Users/qiuyu/www/blog/wp-includes/load.php`
	* 作用: 当访问`http://test.wordpress.com/favicon.ico`这样的URI时, 就只载入了头文件, `header('Content-Type: image/vnd.microsoft.icon');`, 然后就`exit`了. 
12. 执行`wp_maintenance()`函数
	* 位于`/Users/qiuyu/www/blog/wp-includes/load.php`
	* 检查WordPress根目录中的一个文件名为`.maintenance`. 如果文件创建不到10分钟, WordPress*进入维护模式,并显示一条消息。
13. 执行`timer_start()`函数
	* 位于`/Users/qiuyu/www/blog/wp-includes/load.php`
	* 作用: 定义了一个全局变量, `global $timestart`, 记录开始时间. `$timestart = microtime( true )`
14. 执行`wp_debug_mode()`函数
	* 位于`/Users/qiuyu/www/blog/wp-includes/load.php`
	* 根据前面定义的常量: `WP_DEBUG, WP_DEBUG_DISPLAY, WP_DEBUG_LOG, XMLRPC_REQUEST, REST_REQUEST, DOING_AJAX`来决定是否开启debug.
15. 如果前面定义了常量`WP_CACHE`, 就引入缓存文件. 
	* `/Users/qiuyu/www/blog/wp-content/advanced-cache.php"`
16. 执行`wp_set_lang_dir()`函数
	* 位于`/Users/qiuyu/www/blog/wp-includes/load.php`
	* 作用: 获取到语言目录, 然后定义常量`WP_LANG_DIR`和`LANGDIR`.
	* `define( 'WP_LANG_DIR', WP_CONTENT_DIR . '/languages' );`
	* `define( 'LANGDIR', 'wp-content/languages' );`
17. 引入`require( ABSPATH . WPINC . '/compat.php' );`
	* 位于`/Users/qiuyu/www/blog/wp-includes/compat.php`
	* 作用: 处理不用版本的PHP, 函数的差异问题.
18. 引入`require( ABSPATH . WPINC . '/functions.php' );`
	* 位于`/Users/qiuyu/www/blog/wp-includes/functions.php`
	* 作用: wordpress定义的常用函数. 都在这里了.
	* 这里还引入了`require( ABSPATH . WPINC . '/option.php' );`
		* 位置: `/Users/qiuyu/www/blog/wp-includes/option.php`
19. 引入了文件`/class-wp.php`
	* 位置: `/Users/qiuyu/www/blog/wp-includes/class-wp.php`
	* 这个文件有2个类. 
		* `class WP {}`
		* `class WP_MatchesMapRegex {}`
20. 引入了文件`class-wp-error.php`
	* 位置: `/Users/qiuyu/www/blog/wp-includes/class-wp-error.php`
	* 包含了类`class WP_Error {}`
	* 包含了一个函数`is_wp_error( $thing )`
21. 引入了文件`plugin.php`
	* 位置: `/Users/qiuyu/www/blog/wp-includes/plugin.php`
22. 引入了文件`mo.php`
	* 位置: `/Users/qiuyu/www/blog/wp-includes/pomo/mo.php`
		* 这`mo.php`中, 引入了`translations.php`, 
			* 在`translations.php`中引入了`entry.php`文件. 
			* 
		* `mo.php`中, 引入`streams.php`文件.
		*
		*
23. 执行`require_wp_db()`函数
	* 位置: `/Users/qiuyu/www/blog/wp-includes/load.php`
	* 作用: 加载类文件`/Users/qiuyu/www/blog/wp-includes/wp-db.php`, 连接数据库.
24. 执行`wp_set_wpdb_vars`函数
	* 位置: `/Users/qiuyu/www/blog/wp-includes/load.php`
	* 作用: 设置表前缀, 设置表列. 
24. 执行`wp_start_object_cache`函数
	* 位置: `/Users/qiuyu/www/blog/wp-includes/load.php`
	* 作用: 如果在`wp-content`目录中存在缓存文件. 就会使用缓存
25. 未完待续...





