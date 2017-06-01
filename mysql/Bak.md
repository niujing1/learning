###数据备份与还原：

```
mysqldump -uroot -p pass db > /tmp/test.sql
mysqldump -uroot -p pass db tb > /tmp/test.sl 单独备份某张表
mysqldump -uroot -p pass --all-databases > /tmp/test.sql 备份所有数据库
--nodata只转存表结构
--force在转存过程中出错仍继续

2 直接复制整个数据目录，最好在复制前进行lock tables，然后执行flush tables，这样复制数据文件时允许其他客户继续查询表信息，需要使用flush tables来确保开始备份前将所有激活的索引页写入磁盘，但不适用Innodb引擎

3 mysqlhotcopy最快，但只能在同一台机器上操作，且只能用于MyISAM和ARCHIVE引擎
mysqlhotcopy -uroot -ppass db1,db2.. /tmp/test.sql


还原：

1 mysql -uuser -ppass db < /tmp/test.sql
2 或者登陆数据库，执行source /tmp/test.sql
3 直接复制到数据目录，只用于mysql，注意此时权限，要和mysql权限对应
4 mysqlhotcopy快速恢复
```

数据库迁移：

```
相同版本数据库间的迁移：
mysqldump -h www.src.com -uroot -ppass db | mysql -h www.dst.com -uroot -ppass 
--all-databases 迁移全部的数据库

不同版本的数据库之间的迁移
最好使用mysqldump迁移
```

表的导入导出：

```
select column from tb where con into outfile 'filename';
mysql -uroot -p --vertical --execute="select * from test;" test > /tmp/test.sql  --vertical参数显示结果
load data infile '/tmp/test.sql' into table tname [op] [ignore num Lines]
mysqlimport -uroot -ppass db /tmp/test.sql
```

