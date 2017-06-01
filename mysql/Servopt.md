### 服务器性能优化

```
show variables 查看mysql启动之前的系统静态参数
show status 查询服务器运行中的状态信息，比如当前连接数或者锁等待信息
show global status like 'opened_tables'; 曾经打开的表缓存的数
show global status like 'open_tables';当前打开表的缓存数
table_cache同样只适用于MyISAM引擎, 每一个连接进来，都至少会打开一个表缓存，当一个连接访问一个表时，若未缓存，添加缓存，否则直接查询缓存，若缓存已满，会按照一定的规则释放之前的缓存
```


