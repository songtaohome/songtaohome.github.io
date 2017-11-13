---
layout: post
title: redis在php中的应用实例
categories: Linux
description: redis在php中的应用实例
keywords: redis
---

## 安装
开始在 PHP 中使用 Redis 前， 我们需要确保已经安装了 redis 服务及 PHP redis 驱动，且你的机器上能正常使用 PHP。 接下来让我们安装 PHP redis 驱动：下载地址为:https://github.com/phpredis/phpredis/releases。
PHP安装redis扩展
以下操作需要在下载的 phpredis 目录中完成：

```bash
$ wget https://github.com/phpredis/phpredis/archive/2.2.4.tar.gz
$ cd phpredis-2.2.7                      # 进入 phpredis 目录
$ /usr/local/php/bin/phpize              # php安装后的路径
$ ./configure --with-php-config=/usr/local/php/bin/php-config
$ make && make install
```

如果你是 PHP7 版本，则需要下载指定分支：
`git clone -b php7 https://github.com/phpredis/phpredis.git`

修改php.ini文件
`vi /usr/local/php/lib/php.ini`
增加如下内容:

```php
extension_dir = "/usr/local/php/lib/php/extensions/no-debug-zts-20090626"
extension=redis.so
```

安装完成后重启php-fpm 或 apache。查看phpinfo信息，就能看到redis扩展。
![-w486](/images/posts/14881148172490.jpg)


---

## 在php代码中的应用实例

```php
//利用Redis类进行操作

//实例化Redis类
$redis = new Redis();
//选择指定的redis数据库连接，默认端口号为6379
$redis->connect('127.0.0.1', 6379);

//普通set/get操作
$redis->set(‘library’, ‘predis’);
$retval = $redis->get(‘library’);
echo $retval; //显示 ‘predis’

//setex set一个存储时效
$redis->setex(‘str’, 10, ‘bar’); //表示存储有效期为10秒

//setnx/msetnx相当于add操作,不会覆盖已有值
$redis->setnx(‘foo’,12); //true
$redis->setnx(‘foo’,34); //false

//getset操作,set的变种,结果返回替换前的值
$redis->getset(‘foo’,56);//返回34

// incrby/incr/decrby/decr 对值的递增和递减
$redis->incr(‘foo’); //foo为57
$redis->incrby(‘foo’,2); //foo为59

//exists检测是否存在某值
$redis->exists(‘foo’);//true

//del 删除
$redis->del(‘foo’);//true

//type 类型检测,字符串返回string,列表返回 list,set表返回set/zset,hash表返回hash
$redis->type(‘foo’);//不存在,返回none
$redis->set(‘str’,'test’);
$redis->type(‘str’); //字符串，返回string

//append 连接到已存在字符串
$redis->append(‘str’,'_123′); //返回累加后的字符串长度8,此进str为 ‘test_123′

//setrange 部分替换操作
$redis->setrange(‘str’,0,’abc’); //返回3,参数2为0时等同于set操作
$redis->setrange(‘str’,2,’cd’);//返回4,表示从第2个字符后替换,这时’str’为’abcd’

//substr 部分获取操作
$redis->substr(‘str’,0,2);//表示从第0个起，取到第2个字符，共3个，返回’abc’

//strlen 获取字符串长度
$redis->strlen(‘str’); //返回4

//setbit/getbit 位存储和获取
$redis->setbit(‘binary’,31,1);  //表示在第31位存入1,这边可能会有大小端问题?不过没关系,getbit 应该不会有问题
$redis->getbit(‘binary’,31);    //返回1

//keys 模糊查找功能,支持*号以及?号(匹配一个字符)
$redis->set(‘foo1′,123);
$redis->set(‘foo2′,456);
$redis->keys(‘foo*’); //返回foo1和foo2的array
$redis->keys(‘f?o?’);  //同上

//randomkey 随机返回一个key
$redis->randomkey(); //可能是返回 ‘foo1′或者是’foo2′及其它任何一存在redis的key

//rename/renamenx 对key进行改名,所不同的是renamenx不允许改成已存在的key
$redis->rename(‘str’,'str2′); //把原先命名为’str’的key改成了’str2′

//expire 设置key-value的时效性,ttl 获取剩余有效期,persist 重新设置为永久存储
$redis->expire(‘foo’, 1); //设置有效期为1秒
$redis->ttl(‘foo’); //返回有效期值1s
$redis->expire(‘foo’); //取消expire行为

//dbsize 返回redis当前数据库的记录总数
$redis->dbsize();

/*
队列操作
*/

//rpush/rpushx 有序列表操作,从队列后插入元素
//lpush/lpushx 和rpush/rpushx的区别是插入到队列的头部,同上,’x'含义是只对已存在的key进行操作
$redis->rpush(‘fooList’, ‘bar1′); //返回一个列表的长度1
$redis->lpush(‘fooList’, ‘bar0′); //返回一个列表的长度2
$redis->rpushx(‘fooList’, ‘bar2′); //返回3,rpushx只对已存在的队列做添加,否则返回0
//llen返回当前列表长度
$redis->llen(‘fooList’);//3

//lrange 返回队列中一个区间的元素
$redis->lrange(‘fooList’,0,1); //返回数组包含第0个至第1个共2个元素
$redis->lrange(‘fooList’,0,-1);//返回第0个至倒数第一个,相当于返回所有元素,注意redis中很多时候会用到负数,下同

//lindex 返回指定顺序位置的list元素
$redis->lindex(‘fooList’,1); //返回’bar1′

//lset 修改队列中指定位置的value
$redis->lset(‘fooList’,1,’123′);//修改位置1的元素,返回true

//lrem 删除队列中左起指定数量的字符
$redis->lrem(‘fooList’,1,’_'); //删除队列中左起(右起使用-1)1个字符’_'(若有)

//lpop/rpop 类似栈结构地弹出(并删除)最左或最右的一个元素
$redis->lpop(‘fooList’); //’bar0′
$redis->rpop(‘fooList’); //’bar2′

//ltrim 队列修改，保留左边起若干元素，其余删除
$redis->ltrim(‘fooList’, 0,1); //保留左边起第0个至第1个元素

//rpoplpush 从一个队列中pop出元素并push到另一个队列
$redis->rpush(‘list1′,’ab0′);
$redis->rpush(‘list1′,’ab1′);
$redis->rpush(‘list2′,’ab2′);
$redis->rpush(‘list2′,’ab3′);
$redis->rpoplpush(‘list1′,’list2′);//结果list1 =>array(‘ab0′),list2 =>array(‘ab1′,’ab2′,’ab3′)
$redis->rpoplpush(‘list2′,’list2′);//也适用于同一个队列,把最后一个元素移到头部list2 =>array(‘ab3′,’ab1′,’ab2′)

//linsert 在队列的中间指定元素前或后插入元素
$redis->linsert(‘list2′, ‘before’,'ab1′,’123′); //表示在元素’ab1′之前插入’123′
$redis->linsert(‘list2′, ‘after’,'ab1′,’456′);   //表示在元素’ab1′之后插入’456′

//blpop/brpop 阻塞并等待一个列队不为空时，再pop出最左或最右的一个元素（这个功能在php以外可以说非常好用）
//brpoplpush 同样是阻塞并等待操作，结果同rpoplpush一样
$redis->blpop(‘list3′,10); //如果list3为空则一直等待,直到不为空时将第一元素弹出,10秒后超时

/**
set表操作
*/

//sadd 增加元素,返回true,重复返回false
$redis->sadd(‘set1′,’ab’);
$redis->sadd(‘set1′,’cd’);
$redis->sadd(‘set1′,’ef’);

//srem 移除指定元素
$redis->srem(‘set1′,’cd’); //删除’cd’元素

//spop 弹出首元素
$redis->spop(‘set1′);

//smove 移动当前set表的指定元素到另一个set表
$redis->sadd(‘set2′,’123′);
$redis->smove(‘set1′,’set2′,’ab’);//移动’set1′中的’ab’到’set2′,返回true or false

//scard 返回当前set表元素个数
$redis->scard(‘set2′);//2

//sismember 判断元素是否属于当前表
$redis->sismember(‘set2′,’123′); //true or false

//smembers 返回当前表的所有元素
$redis->smembers(‘set2′); //array(’123′,’ab’);

//sinter/sunion/sdiff  返回两个表中元素的交集/并集/补集
$redis->sadd(‘set1′,’ab’);
$redis->sinter(‘set2′,’set1′); //返回array(‘ab’)

//sinterstore/sunionstore/sdiffstore 将两个表交集/并集/补集元素copy到第三个表中
$redis->set(‘foo’,0);
$redis->sinterstore(‘foo’,'set1′); //这边等同于将’set1′的内容copy到’foo’中，并将’foo’转为set表
$redis->sinterstore(‘foo’,array(‘set1′,’set2′)); //将’set1′和’set2′中相同的元素copy到’foo’表中,覆盖’foo’原有内容

//srandmember 返回表中一个随机元素
$redis->srandmember(‘set1′);

/**
有序set表操作
*/

//sadd 增加元素,并设置序号,返回true,重复返回false
$redis->zadd(‘zset1′,1,’ab’);
$redis->zadd(‘zset1′,2,’cd’);
$redis->zadd(‘zset1′,3,’ef’);

//zincrby 对指定元素索引值的增减,改变元素排列次序
$redis->zincrby(‘zset1′,10,’ab’);//返回11

//zrem 移除指定元素
$redis->zrem(‘zset1′,’ef’); //true or false

//zrange 按位置次序返回表中指定区间的元素
$redis->zrange(‘zset1′,0,1); //返回位置0和1之间(两个)的元素
$redis->zrange(‘zset1′,0,-1);//返回位置0和倒数第一个元素之间的元素(相当于所有元素)

//zrevrange 同上,返回表中指定区间的元素,按次序倒排
$redis->zrevrange(‘zset1′,0,-1); //元素顺序和zrange相反

//zrangebyscore/zrevrangebyscore 按顺序/降序返回表中指定索引区间的元素
$redis->zadd(‘zset1′,3,’ef’);
$redis->zadd(‘zset1′,5,’gh’);
$redis->zrangebyscore(‘zset1′,2,9); //返回索引值2-9之间的元素 array(‘ef’,'gh’)
//参数形式
$redis->zrangebyscore(‘zset1′,2,9,’withscores’); //返回索引值2-9之间的元素并包含索引值 array(array(‘ef’,3),array(‘gh’,5))
$redis->zrangebyscore(‘zset1′,2,9,array(‘withscores’ =>true,’limit’=>array(1, 2))); //返回索引值2-9之间的元素,’withscores’ =>true表示包含索引值; ‘limit’=>array(1, 2),表示最多返回2条,结果为array(array(‘ef’,3),array(‘gh’,5))

//zunionstore/zinterstore 将多个表的并集/交集存入另一个表中
$redis->zunionstore(‘zset3′,array(‘zset1′,’zset2′,’zset0′)); //将’zset1′,’zset2′,’zset0′的并集存入’zset3′
//其它参数
$redis->zunionstore(‘zset3′,array(‘zset1′,’zset2′),array(‘weights’ => array(5,0)));//weights参数表示权重，其中表示并集后值大于5的元素排在前，大于0的排在后
$redis->zunionstore(‘zset3′,array(‘zset1′,’zset2′),array(‘aggregate’ => ‘max’));//’aggregate’ => ‘max’或’min’表示并集后相同的元素是取大值或是取小值

//zcount 统计一个索引区间的元素个数
$redis->zcount(‘zset1′,3,5);//2
$redis->zcount(‘zset1′,’(3′,5)); //’(3′表示索引值在3-5之间但不含3,同理也可以使用’(5′表示上限为5但不含5

//zcard 统计元素个数
$redis->zcard(‘zset1′);//4

//zscore 查询元素的索引
$redis->zscore(‘zset1′,’ef’);//3

//zremrangebyscore 删除一个索引区间的元素
$redis->zremrangebyscore(‘zset1′,0,2); //删除索引在0-2之间的元素(‘ab’,'cd’),返回删除元素个数2

//zrank/zrevrank 返回元素所在表顺序/降序的位置(不是索引)
$redis->zrank(‘zset1′,’ef’);//返回0,因为它是第一个元素;zrevrank则返回1(最后一个)

//zremrangebyrank 删除表中指定位置区间的元素
$redis->zremrangebyrank(‘zset1′,0,10); //删除位置为0-10的元素,返回删除的元素个数2

/**
hash表操作
*/

//hset/hget 存取hash表的数据
$redis->hset(‘hash1′,’key1′,’v1′); //将key为’key1′ value为’v1′的元素存入hash1表
$redis->hset(‘hash1′,’key2′,’v2′);
$redis->hget(‘hash1′,’key1′);  //取出表’hash1′中的key ‘key1′的值,返回’v1′

//hexists 返回hash表中的指定key是否存在
$redis->hexists (‘hash1′,’key1′); //true or false

//hdel 删除hash表中指定key的元素
$redis->hdel(‘hash1′,’key2′); //true or false

//hlen 返回hash表元素个数
$redis->hlen(‘hash1′); //1

//hsetnx 增加一个元素,但不能重复
$redis->hsetnx(‘hash1′,’key1′,’v2′); //false
$redis->hsetnx(‘hash1′,’key2′,’v2′); //true

//hmset/hmget 存取多个元素到hash表
$redis->hmset(‘hash1′,array(‘key3′=>’v3′,’key4′=>’v4′));
$redis->hmget(‘hash1′,array(‘key3′,’key4′)); //返回相应的值 array(‘v3′,’v4′)

//hincrby 对指定key进行累加
$redis->hincrby(‘hash1′,’key5′,3); //返回3
$redis->hincrby(‘hash1′,’key5′,10); //返回13

//hkeys 返回hash表中的所有key
$redis->hkeys(‘hash1′); //返回array(‘key1′,’key2′,’key3′,’key4′,’key5′)

//hvals 返回hash表中的所有value
$redis->hvals(‘hash1′); //返回array(‘v1′,’v2′,’v3′,’v4′,13)

//hgetall 返回整个hash表元素
$redis->hgetall(‘hash1′); //返回array(‘key1′=>’v1′,’key2′=>’v2′,’key3′=>’v3′,’key4′=>’v4′,’key5′=>13)

/**
排序操作
*/

//sort 排序
$redis->rpush(‘tab’,3);
$redis->rpush(‘tab’,2);
$redis->rpush(‘tab’,17);
$redis->sort(‘tab’);  //返回array(2,3,17)
//使用参数,可组合使用 array(‘sort’ => ‘desc’,'limit’ => array(1, 2))
$redis->sort(‘tab’,array(‘sort’ => ‘desc’));  //降序排列,返回array(17,3,2)
$redis->sort(‘tab’,array(‘limit’ => array(1, 2)));  //返回顺序位置中1的元素2个(这里的2是指个数,而不是位置),返回array(3,17)
$redis->sort(‘tab’,array(‘limit’ => array(‘alpha’ => true))); //按首字符排序返回array(17,2,3),因为17的首字符是’1′所以排首位置
$redis->sort(‘tab’,array(‘limit’ => array(‘store’ => ‘ordered’))); //表示永久性排序,返回元素个数
$redis->sort(‘tab’,array(‘limit’ => array(‘get’ => ‘pre_*’))); //使用了通配符’*'过滤元素,表示只返回以’pre_’开头的元素

/**
redis管理操作
*/

//select 指定要操作的数据库
$redis->select(‘mydb’); //指定为mydb,不存在则创建

//flushdb 清空当前库
$redis->flushdb();

//move 移动当库的元素到其它库
$redis->set(‘foo’, ‘bar’);
$redis->move(‘foo’, ‘mydb2′); //若’mydb2′库存在

//info 显示服务当状态信息
$redis->info();

//slaveof 配置从服务器
$redis->slaveof(’127.0.0.1′,80); //配置127.0.0.1端口80的服务器为从服务器
$redis->slaveof(); //清除从服务器

//同步保存服务器数据到磁盘
$redis->save();
//异步保存服务器数据到磁盘
$redis->bgsave();
//??
$redis->bgrewriteaof();
//返回最后更新磁盘的时间
$redis->lastsave();

//set/get多个key-value
$mkv = array(
‘usr:0001′ => ‘First user’,
‘usr:0002′ => ‘Second user’,
‘usr:0003′ => ‘Third user’
);
$redis->mset($mkv); //存储多个key对应的value
$retval = $redis->mget(array_keys($mkv)); //获取多个key对应的value
print_r($retval);

//批量操作
$replies = $redis->pipeline(function($pipe) {
$pipe->ping();
$pipe->flushdb();
$pipe->incrby(‘counter’, 10); //增量操作
$pipe->incrby(‘counter’, 30);
$pipe->exists(‘counter’);
$pipe->get(‘counter’);
$pipe->mget(‘does_not_exist’, ‘counter’);
});
print_r($replies);

//CAS,事务性操作

function zpop($client, $zsetKey) {
$element = null;
$options = array(
‘cas’   => true,     // Initialize with support for CAS operations
‘watch’ => $zsetKey, // Key that needs to be WATCHed to detect changes
‘retry’ => 3,        // Number of retries on aborted transactions, after
// which the client bails out with an exception.
);

$txReply = $client->multiExec($options, function($tx)
use ($zsetKey, &$element) {
@list($element) = $tx->zrange($zsetKey, 0, 0);
if (isset($element)) {
$tx->multi();     // With CAS, MULTI *must* be explicitly invoked.
$tx->zrem($zsetKey, $element);
}
});
return $element;
}
$zpopped = zpop($redis, ‘zset’);
echo isset($zpopped) ? “ZPOPed $zpopped” : “Nothing to ZPOP!”, “\n”;

//对存取的key加前缀,如: ‘nrk:’
$redis->getProfile()->setPreprocessor(new KeyPrefixPreprocessor(‘nrk:’));

//分布式存储的一些方法
$multiple_servers = array(
array(
‘host’     => ’127.0.0.1′,
‘port’     => 6379,
‘database’ => 15,
‘alias’    => ‘first’,
),
array(
‘host’     => ’127.0.0.1′,
‘port’     => 6380,
‘database’ => 15,
‘alias’    => ‘second’,
),
);

use Predis\Distribution\IDistributionStrategy;

class NaiveDistributionStrategy implements IDistributionStrategy {
private $_nodes, $_nodesCount;

public function __constructor() {
$this->_nodes = array();
$this->_nodesCount = 0;
}

public function add($node, $weight = null) {
$this->_nodes[] = $node;
$this->_nodesCount++;
}

public function remove($node) {
$this->_nodes = array_filter($this->_nodes, function($n) use($node) {
return $n !== $node;
});
$this->_nodesCount = count($this->_nodes);
}

public function get($key) {
$count = $this->_nodesCount;
if ($count === 0) {
throw new RuntimeException(‘No connections’);
}
return $this->_nodes[$count > 1 ? abs(crc32($key) % $count) : 0];
}

public function generateKey($value) {
return crc32($value);
}
}

//配置键分布策略
$options = array(
‘key_distribution’ => new NaiveDistributionStrategy(),
);

$redis = new Predis\Client($multiple_servers, $options);

for ($i = 0; $i set(“key:$i”, str_pad($i, 4, ’0′, 0));
$redis->get(“key:$i”);
}

$server1 = $redis->getClientFor(‘first’)->info();
$server2 = $redis->getClientFor(‘second’)->info();

printf(“Server ‘%s’ has %d keys while server ‘%s’ has %d keys.\n”,
‘first’, $server1['db15']['keys'], ‘second’, $server2['db15']['keys']

```






