### MySQL 锁机制

InnoDB是行锁、MyISAM和MEMORY是表锁、BDB是页锁

行锁：

```
锁定粒度小，发生锁争用的概率小，可以给程序最大的并发能力，但，锁开销大，加锁慢，易产生死锁
行锁定不是MySQL实现的，而是InnoDB存储引擎自己实现的
共享锁和排他锁
```

表锁：

```
实现简单、资源消耗小，不易发生死锁
发生锁冲突的概率大，并发较低
MySQL的表级锁定分为读锁和写锁，MySQL提供了四个队列来维护，
Current read lock queue(lock -> read)
Padding read lock queue(lock -> read wait)
Current write lock queue(lock -> write)
Padding write lock queue(lock -> write wait)
session1加读锁  1、2会话可读
加写锁，其它会话不可读
```

获取MyISAM表级锁的争用情况：

```
show status like "table"\G;
table_locks_immediate: 产生表级锁定的次数
table_locks_waited: 出现表级锁定而发生等待的次数, 若table_locks_waited的值比较高，说明存在了严重的锁争用
```

MyISAM的性能分析：

```
MyISAM早读操作占主导的情况下是高效的，一旦出现大量的读写并发，同InnoDB相比，执行效率就会出现严重的下降
对于MyISAM存储引擎表，新的数据会被附加到数据文件的结尾，可如果经常做一些update和delete操作，新的数据会是不连续的，数据文件会出现很多空洞，此时再插入，会先检查这些空洞是否可以容纳新的数据，如果可以，新的数据会写入到空洞，否则插入到文件尾，这样做是为了减少产生文件碎片的可能性
MyISAM往往会因为读表请求的增加，出现严重的读写锁问题，所以，常常采用读写分离
此时会出现另一个问题：从服务器上大量的查询操作会被来自主从同步的insert和update操作严重阻塞，造成从库负载迅速上升，在解决读写互斥的问题上，由于没有办法在短期内增加读服务器，所以，只能通过MySQL配置牺牲数据实时性来保证服务器的安全
1）concurrent_insert=2，不管表有没有空洞，允许表结尾并发插入，产生的碎片文件可以通过optimize_table优化
2）默认写优先级高于读，即使先发送的是读请求，后发送的是写请求，也会优先处理写请求，可以考虑设置max_write_lock_count=1，此时当系统处理一个写操作后，就会暂停写操作，给读机会
3）给读较高的优先级 设置low-priority-updates=1
```

InnoDB锁分析：

```
共享锁：lock in share mode
不同进程可以查询，不可以更新
排他锁： select for update 不同进程不可以查询
```

查询InnoDB锁争用的情况：

```
show status like "%innodb_row_lock%";
Innodb_row_lock_current_waits：当前等待的锁数量
Innodb_row_lock_time：系统启动以来锁定总时间
Innodb_row_lock_time_avg：每次锁等待平均花费的时间
Innodb_row_lock_waits：系统启动到现在的锁等待的次数
```

注意：InnoDB的行级锁是通过给索引加上索引项加锁来实现的，InnoDB的行级锁只有通过索引条件检索数据才可以使用行级锁，否则使用表锁, 行锁，在非锁定行上可以执行操作

间隙锁：

```
在更新InnoDB某个区间的数据时，会锁定这个区间的所有记录
update filed where id between 1 and 10，则1到10之间所有的记录都会被锁定，就算是记录不存在也会锁定

select @@tx_isolation; 查看数据库的隔离级别
一般来说，数据库的隔离级别越高，锁越严格，锁冲突的可能性也就越大
set session transaction isolation level repeatable read; 设置隔离级别
```

死锁：

```
一般死锁的解决是一个事务回滚释放锁，另一个事务继续完成事务，但在涉及外部锁，或表锁的情况下，InnoDB并不能完全自动检测到死锁，需要通过设置等待时间(innodb_lock_wait_timeout)来解决，通常情况下都是英语涉及问题，通过调整业务流程，事务大小，数据库访问的SQL，大多数死锁都可以避免
```

InnoDB锁优化建议：

```
1）应尽量控制事务的大小，减少锁定资源量和锁定时间
2）让数据检索通过索引完成，避免行锁变成表锁
3）减少基于范围的检索过滤条件
4）在业务环节允许的情况下，尽量使用较低级别的事务隔离，减少因事务隔离代理的附加成本
5）合理使用索引，让InnoDB在加锁时更加精准
6）在应用中尽可能按照相同的访问顺序来访问，防止产生死锁
7）尽可能做到一次锁定所需的全部资源，防止产生死锁
8）对容易产生死锁的业务，放弃行锁，考虑表锁
9）不要申请超过实际需要的锁级别
```






