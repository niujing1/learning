### 高级特性

查询缓存：

查看查询缓存状态
```
show variables like '%query_cache%';
临时修改设置： 
SET GLOBAL var_name = value;  
SET @@GLOBAL.var_name = value;  
SET SESSION var_name = value;  
SET @@SESSION.var_name = value;

永久修改可以修改my.cnf添加配置项：
query_cache_size=value
```

```
show status like "Qcache_hits" 可以查看缓存命中
在执行更新之后第一次查询缓存不会被命中，所以不太时候频繁更新的表
```

监控和维护查询缓存：

```
flush query cache 整理查询缓存，以更好的利用，它不会从缓存中清除查询结果
reset query cache 用于移除查询缓存中所有的查询结果
show status like "Qcache%" 可以查看查询缓存的使用状态
```

提高缓存命中率：

```
1）客户端、服务端字符集保持一致
2）SQL不区分大小写，但对于缓存来说会当成不同的SQL来执行
3）查询缓存不存储不确定结果，任何一个包含不确定函数的查询不会被缓存，eg now
4）查询缓存只发生在服务器第一次接到SQL查询语句时，对查询中的子查询、视图查询、和存储过程查询都不能缓存
使用了临时表的查询也不会被缓存
```

使用查询缓存

```
优：改善了MySQL服务器的性能，使查询更加高效
缺：查询缓存本身需要消耗系统IO资源，也增加了服务器的额外开销
1）在查询之前要检测缓存中是否有对应条目
2）若无，会把查询结果缓存到查询缓存，也需要额外消耗资源
3）insert操作发送时，查询缓存无效，同样需要消耗系统资源
```


一旦发生update或者insert查询缓存会失效
可以采用分区表提高缓存命中率


优化查询缓存：

```
1）设计数据库的时候表尽量小
2）尽量一次性插入
3）尽量不要在数据库或者表的基础上控制查询缓存，可以使用SQL_CACHE控制(query_cache设置为2 查询语句使用query_cache才会有效)
4）对于有很多写任务的程序、关闭查询缓存更好
5) 禁用查询缓存可以使query_cache_size的值设置为0，这样就不会消耗任何内存
6）若想部分使用，可以将全局变量query_chache_type的值设置为2(DEMAND)，然后再需要使用查询缓存时，在语句后边加上SQL_CHACHE，不需要的时候加上QUERY_NO_CHACHE
```

合并表和分区表：

```
合并表本质上是根据union产生的表的联合

分区表是把一张大的表拆分成多个小表的过程，为了减少某些特定查询的时间，主要有垂直分区和水平分区

水平分区是根据行记录拆分、以表的某个属性作为分割点，比如时间大于2016-12-01 的
垂直分区是把列拆分，减少列宽度

1. range 分区
create table tmp(empno varchar(20) not null, empname varchar(20), deptno int, birthdate date, salary int) partition by range(salary)(partition p1 values less than(1000), partition p2 values less than(2000)， partition p4 values less than maxvalue); 其中maxvalue表示可能的最大整数值

2. list分区，类似range分区，但是list分区的值从属与一个集合、range分区的值从属与一个区间
create table tmp(empno varchar(20) not null, empname varchar(20), deptno int, birthdate date, salary int) partition by list(deptno)(partition p1 values in(10, 20), partition p2 values in (30));

3. hash分区，基于用户定义的表达式的返回值来选择分区，搞表达式使用将要插入到表中的这些行的列值进行计算，这个函数可以包含MySQL中有效的、产生非负数的任何表达式，主要是为了保证数据的平均分布
create table tmp(empno varchar(20) not null, empname varchar(20), deptno int, birthdate date, salary int) partition by hash(year(birthday)) partitions 4;

4. 线性hash分区，它与hash分区的区别在于：它使用的是一个线性的2的幂运算法则，而hash分区使用的是hash函数的模数 partition by linear hash

5. key分区，类似于hash分区，但key分区只支持计算1列或者多列，且MySQL服务提供自身的hash函数 partition by key(birthdate)

6. 复合分区，是对分区表中的每个分区再次分割、子分区既可以使用hash分区，也可以使用key分区
range-hash 、range-key 、 list-hash 、 list-key
MyISAM/InnoDB/NDB都支持分区、CSV、MERGE不支持分区
```

适用场景：

```
1、表非常大无法全部放到内存，或者只有部分热点数据，其它均为历史数据
2、分区表更容易维护(可独立对分区进行优化，检查、修复以批量删除大数据可以采用drop分区的形式)
3、分区表的数据可以分布在不同的物理设备上、从而搞笑地利用多个硬件设备
4、分区表可以避免某些特殊的瓶颈(InnoDB的单个索引的互斥访问，ext3文件系统的inode锁竞争等)
5、可以独立的恢复和备份独立的分区，适用于大数据集的场景
```

分区表的限制：

```
1、单表只支持1024个分区
2、MySQL5.1只能对数据表的整型列进行分区。或者数据列可以通过分区函数转化为整型列，MySQL5.5的Range、list类型可以直接使用列分区
3、如果分区字段中有主键或唯一索引的列，那么所有的主键列和唯一索引列都必须包含进来
4、分区表无法使用外键约束
5、分区必须使用相同的Engine
6、对于MyISAM分区表，不能在使用LOAD INDEX INTO CACHE操作
7、对于MyISAM分区表，使用时会打开更多的文件描述符（单个分区是一个独立的文件）
```

分区策略：

```
1. 全量扫描数据，不需要任何索引：通过where条件大概定位到哪个分区，必须将查询所需要扫描的分区个数限制到最小数量
2. 建立分区索引、分离热点
```

分区应该注意的点：

```
1. NULL值会使分区过滤无效:
2. 重组分区或者类似alter语句可能会造成很大的开销
新建或者删除分区操作很快，重组分区或者类似ALTER语句操作会先创建一个临时的分区，将数据复制其中，然后在删除原分区
3. RANGE类型分区随着分区数量增加会对MYSQL额外增加查询分区定义列表（符合条件行在哪个分区）的压力，尽量限制适当的分区数量;key和hash类型分区不存在此问题
```

事务控制：

```
1. 可以在读锁之上加读锁，不可以在读锁之上加写锁，也不能在写锁之上加写锁
2. 分布式事务，就是对多个事务的管理，保证多个MySQL数据库同时提交或者回滚
1) 内部分布式：binlog作为协调者，用于同一实例下跨多个数据引擎的事务
2）外部分布式：应用层介入作为协调者，用于跨多个实例的分布式，判断全部提交或者回滚
```






```

