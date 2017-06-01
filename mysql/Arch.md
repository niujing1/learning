### MySQL架构


MySQL各个模块的特点：

```
1. 客户端通过连接/线程处理层来连接mysql数据库，连接/线程处理层主要用来处理客户端的请求、身份验证和数据库安全验证等
2. 查询缓存和查询分析器是SQL层的核心部分，主要涉及查询的解析、优化、缓存以及所有的内置函数、存储过程、触发器、视图等功能
3. 优化器主要负责存储和获取所有存储在MySQL中的数据
这3层可以统称为SQL层

上边有客户端、下边有存储引擎层
```

MySQL物理文件的组成：

1. 日志文件

```
1）错误日志：默认关闭、--log-error选项可以在启动的时候指定，也可以写到配置文件
2）二进制日志：BinaryLog也就是binlog 记录了mysql所有修改数据库的操作、然后以二进制的形式记录在日志中，还包括每条SQL执行的时间和消耗的资源，及相关的事务信息
3）慢查询日志：超过long_query_time的设置的语句，可以通过log-slow_queries(5.6之前)、slow_query_log_file之后
4）InnoDB引擎在线Redo日志：它记录了InnoDB所作的所有物理变更和事务信息,文件以ib_logfile[num]来命名，日志目录可以用innodb_log_group_home_dir来控制,innodb_log_files_in_group可以设置日志的数量

```

2. 数据文件

```
每一个数据库都会在数据文件夹下以单独的文件夹存放

.frm 文件：无论是哪种存储引擎，都会生成一个表名为名称的frm文件，主要存放与表相关的信息，包括表结构定义信息，数据库崩溃时可以用它恢复

.MYD文件：MyISAM创建表时用它用它来存放数据

.MYI文件：存放表数据文件中任何索引的数据树，可以被缓存的文件主要源于MYI文件

.ibd 和 .ibdata: innodb存储引擎的数据，主要包括索引信息，InnoDB采用这两种存储方式，主要是因为可以通过配置来决定是采用共享表空间还是独享表空间
若采用共享表空间，会采用ibdata来存储，所有的表使用一个或者多个ibdata
独享表空间会采用ibd文件
共享表空间通过innodb_data_home_dir和innodb_data_file_path(配置文件路径及文件名称)共同配置
如果需要添加新的ibdata文件，则需要在innpdb_data_file_path参数后边配置，然后重启服务才会生效
show binary logs\G; 查看所有的bin-log日志
purge binary logs to 'mysql-bin.000004';删除掉4之前的所有bin-log日志
再通过show binary logs\G;查看，可以看到bin-log被删除了
show binlog events; 查看binlog记录的信息

.pid文件存储进程ID
 
```

MySQL各逻辑块联系：

```
可以通过mysqld --verbose --help来查看当前系统所有的配置值
MySQL启动之后，MySQL的初始化模块从系统配置文件读取系统参数和命令参数，并按照参数初始化整个系统，同时存储引擎也在启动，当初始化结束之后，连接管理模块会监听并接受客户端的程序，连接模块会将连接请求转发给线程管理模块去请求一个连接线程，线程管理收到请求会调用用户模块进行授权检查，通过后检查线程池是否有可用线程连接，若有就取出跟客户端连接上，若无，则建立一个新的线程连接
MySQL中的请有2种，1是需要命令解析和分发模块才能执行请求的操作，另一种是不需要转发就可以直接执行。另外此时会记录到日志
Query型的请求，会将控制权交给Query解释器，Query解析器会检查是否是select类型的请求，若是，启动缓存模块，若已有结果集，直接将缓存数据通过线程传给客户端，若无，则通过查询分发器给相应的处理模块
若是DML或者DDL则交给变更管理模块；若是检测修复类查询交给表维护模块；若是未被缓存的查询，交给查询优化器，实际上表变更模块又分为了insert处理器、delete处理器等小的模块，等命令全部执行结束，控制权交还给连接线程模块

复制模块：分为master模块和slave模块，master模块负责读取Master端的binary日志以及slave端的IO线程交互，slave模块有两个线程，一个负责从master请求和接收binary日志，写入本地IO线程，另一个线程从relay log读取日志事件，解析成可以在slave端执行的命令，交给Slave端的SQL进程
```

####MySQL存储引擎
支持的存储引擎大概有: MyISAM、MEMORY、InoDB、BLACKHOLE、EXAMPLE、ARCHIVE、CSV、等
InnoDB的批量插入支持度要低于MyISAM，但是内存使用和空间使用率比较高，支持行锁

```
show engines \G; 查看当前数据库支持的引擎
``` 

MyISAM引擎：

```
不支持事务、不支持主键、对数据的存储和批量查询速度比较快，对于不需要事务，主要以查询和增加记录为主的应用采用MyISAM引擎，MyISAM引擎只缓存索引，对数据文件采用操作系统缓存，若索引数据文件超过分配得到缓存空间时，也会采用操作系统缓存,但可以支持表压缩
若在存储过程中发生错误，可以通过check table语句检测myisam表的状态、然后用repair table来修复，也可以通过myisamchk xx.MYI来检测

MyISAM表有3种不同的存储方式，静态表、动态表、压缩表
静态表和动态表数据存储会根据数据列的类型自动选择，压缩表只能通过myisampack创建
静态表中的字段都是定长的，每条记录都是固定长度的，它存储速度非常快，并且容易缓存，表发生缺损后容易修复，缺点是所用的空间会比动态表多
myisampack可以压缩MYISAM表，每行可以单独压缩、每列的压缩也不一样
```

InnoDB存储引擎：

```
支持事务、支持列自增、支持外键
```

MEMORY表：

```
执行速度最快，每个memory表实际上和一个磁盘文件关联，文件名采用.frm的格式，默认采用hash索引，访问速度最快，但是，一旦数据库发生故障，内存中的数据就会丢失，表数据量变大时会自动转化为磁盘表，可以通过max_heap_size来设置，默认16M，对单个表也可以指定max_rows指定最大行数
```

Merge表会进行表数据合并

MySQL命令行实用工具：

```
mysqld: SQL后台程序(即MySQL服务器进程)，它运行之后，客户端才能通过连接服务器访问数据库
mysqld_safe：服务器启动脚本,增加了一些安全特性，发生错误时，向错误日志写入信息
mysqld.server：访问启动脚本，调用mysqld_safe
mysql_multi:访问启动脚本，可以检测、启动系统安装的多个服务器
mysqlbug：MySQL缺陷报告脚本
mysql_install_db:只在安装时运行一次，用默认权限创建MySQL授权表

MySQL客户端常用工具：
myisampack：压缩myisam以产生更小的一个只读表
mysqlaccess：检测访问主机名、用户名和数据库组合权限的脚本
mysqlbinlog：从二进制日志读取语句的工具，在二进制日志文件中包含执行过的语句，可以帮助系统从崩溃中恢复
mysqlhotcopy:在mysql运行时快速备份myisam表的工具
select current_user()\G;可以查看当前所有用户

mysqlbinlog mysql-bin.xxx可以读取mysqlbinlog日志
mysqlbinlog mysql-bin.xxx -d --start-time='XXX' -d只列出跟数据库相关的操作，--start-time指定开始时间 --stop-time指定结束时间
mysqlcheck -A检测所有数据库，-analyze(-a)分析指定数据库
-auto-repair若检测的数据库发生损坏，自动修复
-check(-c)检测错误
-repair修复
-optimize(-o)指定优化数据库
```
