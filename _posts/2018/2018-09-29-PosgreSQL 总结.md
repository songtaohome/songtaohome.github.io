---
layout: post
title: PosgreSQL 总结
categories: PosgreSQL
description: PosgreSQL 总结
keywords: PosgreSQL, 学习
---

## PosgreSQL 简介

* PosgreSQL是一个关系型数据库管理系统（ORDBMS）

* 从加州大学伯克利分校写的Postgres软件包发展而来的。目前为了发音方便，依旧沿用了这个名称。

## Why PosgreSQL (PosgreSQL相对于MySQL的优势列举)

* PostgreSQL拥有很强大的查询优化器,支持很复杂的查询处理。

* MySQL不支持函数索引,只能在创建基于具体列的索引。

* PostgreSQL表增加列,只是在数据字典中增加表定义,不会重建表。

* PostgreSQL支持包括PL/pgSQL、PL/Java、PL/Perl、PL/php、PL/Python、PL/R、PL/Ruby、PL/sh、PL/Tcl与PL/Lua等编程语言。MySQL存储过程与触发器的功能有限。可用来编写存储过程、触发器、计划事件以及存储函数的语言功能较弱。

* MySQL只有一种表连接类型:嵌套循环 连接(nested-loop),不支持排序-合并连接(sort-merge join)与散列连接(hash join)。而PostgreSQL都支持。

* InnoDB的表和索引都是按相同的方式存储。也就是说表都是索引组织表。这一般要求主键不能太长而且插入时的主键最好是按顺序递增,否则对性能有很大影响。

* PostgreSQL提供了视图功能,内置的性能视图可以方便的看到发生在一个表和索引上的select、delete、update、insert统计信息以及缓存命中率命中率。

* MySQL支持的SQL语法(ANSI SQL标准)的很小一部分。不支持递归查询、通用表表达式(Oracle的with语句)或者窗口函数(分析函数)。

* 类似于ALTER TABLE或CREATE TABLE一类的操作(DDL)都是非事务性的.它们会提交未提交的事务,并且不能回滚也不能做灾难恢复。PosgreSQL中DDL也是有事务的。

## PosgreSQL jsonb结构

* sql操作符

```
-> 返回类型必须为object类型
   右接int类型：表示去数组元素的角标对应的值。
   右接text类型：表示取jsonb中key对应的值
   示例：select '[{"a":"foo"},{"b":"bar"},{"c":"baz"}]'::jsonb -> 2

->> 返回text类型，object类型会转为text类型返回
   右接int类型：表示去数组元素的角标对应的值。
   右接text类型：表示取jsonb中key对应的值
   示例：select '{"a": {"b":"foo"}}'::jsonb ->> 'a'

#> 右接text[]类型，返回类型必须为object类型
   示例：select '{"a": {"b":{"c": "foo"}}}'::jsonb #> '{a,b}'

#>> 右接text[]类型，返回text类型，object类型会转为text类型返回
   示例：select '{"a":[1,2,3],"b":[4,5,6]}'::jsonb #>> '{a,2}'

? 右接text类型，判断jsonb中是否包含某个key，返回true or false
   示例：select '{"a":1, "b":2}'::jsonb ? 'b'

?| 右接text[]类型，判断jsonb中key与数组中的元素是否有交集，返回true or false
   示例：select '{"a":1, "b":2, "c":3}'::jsonb ?| array['b', 'c']
   
?& 右接text[]类型，判断jsonb中key是否包含数组中的所以元素，返回true or false

|| 右接jsonb类型，将右接的jsonb合并的jsonb中。(也可用于字符串相加)
   示例： select '{"a":"1"}' || coalesce(null,'{}'::jsonb);

- 表示删除jsonb中的元素
   右接text类型，将单个key对应的k/v删除
   右接text[]类型，将数组中所有key对应的k/v删除
   右接integer类型，将数组角标对应的元素删除

#- 右接text[]，表示深层删除jsonb中的元素
   示例：select '["a", {"b":1}]'::jsonb #- '{1,b}'

@> 右接jsonb类型，判断jsonb中是否包含右接的jsonb，返回true or false
   示例：select '{"a":1, "b":2}'::jsonb @> '{"b":2}'::jsonb

<@ 右接jsonb类型，判断jsonb字段中是否包含于右接的jsonb，返回true or false
```

* 索引jsonb下的字段

```
jsonb下的字段也可通过操作符进行指定并添加到索引：
    示例：CREATE INDEX comment_find_parents_update
        ON public.comment
        USING btree
        (invitation, info ->> 'language', update_at DESC)
        WHERE is_spam = false AND is_deleted = false AND parent = ARRAY[]::integer[];
```

## PosgreSQL 索引与优化

* PosgreSQL 索引

```
PostgreSQL只支持堆表,不支持索引组织表, Innodb只支持索引组织表

  索引组织表的优势:
  表内的数据就是按索引的方式组织,数据是有序的,如果数据都是按主键来访问,那么访问数据比较快。
  而堆表,按主键访问数据时,是需要先按主键索引找到数据的物理位置。

  索引组织表的劣势:
  索引组织表中上再加其它的索引时,其它的索引记录的数据位置不再是物理位置,而是主键值,所以对于索引组织表来说,
  主键的值不能太大,否则占 用的空间比较大。
  对于索引组织表来说,如果每次在中间插入数据,可能会导致索引分裂, 索引分裂会大大降低插入的性能。
  所以对于使用innodb来说,我们一般最好让主键是一个无意义的序列,这样插入每次都发生在最后,以避免这个问题。
  索引组织表的全表扫描要比堆表慢。
```
* SQL 索引优化原则
  
```
如果与一个查询相关的索引行是相邻的，或者至少相距足够靠近的话，这最小化了必须扫描的索引片的宽度。
如果一个索引行的顺序与查询语句的需求一致，这消除了排序操作。
如果一个索引行包含查询语句中的所有列，这避免了访问表的操作。
```
  
* PosgreSQL 索引优化
  
  
```
添加部分索引，命中部分索引需要条件中的所有项都匹配。这可以减小索引的尺寸，同时也将加速使用索引的查询。
它也将加速很多表更新操作，因为这种索引并不需要在所有情况下都被更新。

组合索引，PosgreSQL有组合多个索引（包括多次使用同一个索引）的能力来处理那些不能用单个索引扫描实现的情况。
为了组合多个索引，系统扫描每一个所需的索引并在内存中准备一个位图用于指示表中符合索引条件的行的位置。
然后这些位图会被根据查询的需要“与”和“或”起来。最后，实际的表行将被访问并返回。表行将被以物理顺序访问，
因为位图就是以这种顺序布局的。这意味着原始索引中的任何排序都会被丢失，并且如果存在一个ORDER BY子句就需要
一个单独的排序步骤。由于这个原因以及每一个附加的索引都需要额外的时间，即使有额外的索引可用，规划器有时也会
选择使用单一索引扫描。
```
  
##  PosgreSQL 缺点
  
*  由于索引中完全没有版本信息,不能实现Coverage index scan,即查询只扫描索引,直接从索引中返回所需的属性,还需要访问表。对于类似select count(*) from table的语句,PostgreSQL会比较慢。

* 需要对vacuum做仔细的安排,特别对于更新频繁的数据库vacuum是由PostgreSQL多版本设计决定的。vacuum不能回收表已经占用的空间。

* 不支持裸设备，要求OS下有一个健壮的文件系统。在Linux下我们一般选XFS,对于solaris下选ZFS。oracle对裸设备有很好的支持,而MySQL的innodb引擎也可以放在裸设备下,但由于独享表空间必需是一个文件一张表,所以使用裸设备的管理成本也比较高。

## PosgreSQL 资源列表
[PosgreSQL 中文社区](http://www.postgres.cn/index.php/home)

[PosgreSQL 官网](https://www.postgresql.org/)
  
