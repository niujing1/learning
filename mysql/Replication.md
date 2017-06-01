### MySQL复制

MySQL的复制操作：

```
1. 主服务器将数据的改变记录到二进制日志中(binary log)
2. 从服务器将主服务器的binlog复制到它的中继日志中 relay-log
3. 从服务器重做主服务器的事件，将数据的改变保持与主服务器同步
```

Linux下的主从复制：

安装mysql：

```
1. 下载安装MySQL
2. 创建mysql安装程序的目录和数据文件的目录
   mkdir -p /usr/local/mysql
   mkdir -p /usr/local/mysql/cata
   groupadd mysql
   useradd -r -g mysql mysql
3. 解压源文件
   tar -zxvf mysql-5.6.tar.gz
   cd mysql-5.6
   cmake . #编译源码
4. 修改MySQL安装目录的权限
   chown -R mysql.mysql /usr/local/mysql
5. 安装源码
   cd /usr/local/mysql/scripts
   ./mysql_install_db --user=mysql --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data
   cd /usr/local/mysql/support-files
   cp mysql.server /etc/rc.d/init.d/mysql
   cp my-default.cnf /etc/my.cnf
   chkconfig --add mysql
   chkconfig mysql on
6. 启动mysql
   service mysql start
```

使用mysql_multi管理多个mysql实例

```
1. service mysql stop #关闭正在运行的mysql进程
2. lsof -i :3306 检测端口是否已释放，mysql是否成功关闭
3. 把常用工具添加到/usr/bin下
   ln -s /usr/local/mysql/bin/mysqld_multi /usr/bin/mysqld_multi
   ln -s /usr/local/mysql/bin/mysqld_install_db /usr/bin/mysql_install_db
4. 初始化数据目录并安装3个mysql服务
   cd /usr/local/mysql
   mkdir -p /usr/local/var/mysql1
   mkdir -p /usr/local/var/mysql2
   mkdir -p /usr/local/var/mysql3
5. 从mysql的源码中把mysql.server复制到/etc/init.d目录下，然后执行
   cp my-default.cnf /etc/my.cnf

6. 配置/etc/my.cnf

#The mysql server
[mysqld_multi]
mysqld = /usr/local/mysql/bin/mysqld_safe
mysqladmin = /usr/local/mysql/bin/mysqladmin
user = root

[mysqld1]
port = 3306

[mysqld2]
port = 3307
socket = /tmp/mysql2.sock
datadir = /usr/local/var/mysql2

[mysqld3]
port = 3308
socket = /tmp/mysql3.sock
datadir = /usr/local/var/mysql3

[mysqld]
...

7. 查看数据库的状态
mysqld_multi --defaults-extra-file=/etc/my.cnf report

8. 使用mysqld_multi启动mysql服务
   mysqld_multi --defaults-extra-file=/etc/my.cnf stop
   mysqld_multi --defaults-extra-file=/etc/my.cnf start
   mysqld_multi --defaults-extra-file=/etc/my.cnf report

9. 登录数据库
   mysql -uroot -p -P3306
   mysql -uroot -p -P3307
   mysql -uroot -p -P3308
   show variables like 'socket' 查看数据库的pid和socket状态
   此时发现3个数据库的pid和socket都一样，可以使用mysql -u root -S /tmp/mysql2.sock用-S参数来指定然后登录查看可以发现pid和socket文件都已经改变
10. 重启数据库
   mysqld_multi --defaults-extra-file=/etc/my.cnf stop 1-3
   mysqld_multi --defaults-extra-file=/etc/my.cnf stsrt 1-3
```

mysqld_multi实现单机主从复制：

```
mysql的复制至少需要两个mysql服务，可以使用不同的服务器或者再一台服务器上启动多个mysql服务
主要有3个线程：
其中两个线程(SQL线程和IO线程)在slave端，一个线程(IO线程)在master端，要实现复制过程，master必须打开binlog，复制其实就是slave从master端获取binlog日志，然后在自己服务器上顺序执行日志中记录的各种操作

1. 使用mysqld_multi开启三个已经设定好的mysql服务，执行：
mysqld_multi --defaults-extra-file=/etc/my.cnf stsrt 1-3
2. 登录mysql主服务器，设置一个复制使用的账户repl，授予replication slave权限
grant replication slave on "*"."*" to 'repl'@localhost identified by '123'
grant replication slave on "*"."*" to 'repl'@'%' identified by '123'

3. 修改主服务器的配置文件my.cnf，开启binlog，并设置server-id的值
  vi /etc/my.cnf
  [mysqld1]
  port = 3306
  log-bin = /usr/local/var/mysql1/mysql-bin
  server-id = 1

  然后使用mysqld_multi重启服务
  在master服务器设置读锁定有效，防止数据库操作使数据不一致
  登录mysql 执行： flush tables with read lock
4. 查看日志情况  show master status\G;
5. 主服务器此时可以做一个备份，可以在服务器停止的情况下支架使用系统复制命令
   tar -cvf data.tar data
6. 主数据库备份完成之后，主库可以恢复读写操作
   unlock tables;
7. master主服务器已经配置成功，如果主服务器mysqld选项设置server-id选项，但从服务器没有设置server-id参数，启动从服务器会发生以下错误：
mysql> start slave
ERROR 1200 (HY000): The server is not configured as slave; fix in config file or with CHANGE MASTER TO
   继续编辑my.cnf
   [mysqld1]
   port = 3306
   log-bin = /usr/local/var/mysql1/mysql-bin
   server-id = 1
   
   [mysqld2]
   port = 3307
   socket = /tmp/mysql2.sock
   datadir = /usr/local/var/mysql2
   log-bin = /usr/local/var/mysql2/mysql-bin
   server-id = 2

   [mysqld3]
   port = 3308
   socket = /tmp/mysql3.sock
   datadir = /usr/local/var/mysql3
   log-bin = /usr/local/var/mysql3/mysql-bin
   server-id = 3

8. 重启mysql主服务器
   mysqld_multi --defaults-extra-file=/etc/my.cnf stop 1-3
   mysqld_multi --defaults-extra-file=/etc/my.cnf stsrt 1-3
   mysqld_multi --defaults-extra-file=/etc/my.cnf report
9. 对从服务器做相应的设置，此时需要制定复制使用的用户，主数据的ip地址，端口以及开始复制的日志文件和位置等，具体设置如下：
   mysql -uroot -p -P3307 -S /tmp/mysql2.sock
   登录之后执行： show variables like '%log-bin%'
   mysql> stop slave
   mysql> change master to master_host='127.0.0.1' master-user='repl' master-passwor='123' master-log-file='mysql-bin.000001' master-log-pos=120
   mysql> start slave;
   mysql> show slave status\G;
可以发现从服务器以及设置成功，此时也可以执行show processlist\G命令查看从服务器的进程状态
10. 在master上执行insert，然后在从服务器上查看数据是否可以正确同步
```

不同服务器之间实现同步：

```
1. 确保主从服务器的mysql版本一致
2. 创建一个复制使用的账户rep1并授权
mysql> grant replication slave on  "*"."*" to 'repl'@'192.168.1.101' identified by '123'
3. 修改主服务器配置文件，开启binlog，并设置server-id的值
   [mysqld1]
   port = 3306
   log-bin = /usr/local/var/mysql1/mysql-bin
   server-id = 1
4. 在主服务器上设置读锁定有效，为了确保没有数据库操作，以便获得一致性的快照
   flush tables with read lock
5. 查询主服务器上当前二进制日志名和偏移值，为了在从服务器启动以后从这个点开始数据库的恢复
   mysql> show master status;
6. 停止主服务器更新操作，需要生成数据库的备份，可以通过mysqldump导出数据或者使用ibackup进行数据库的备份，如果主数据库停止，可以直接cp数据文件到从实践看服务器上
7. 主库备份完成之后，恢复读写操作
   mysql> unlock tables;
8. 修改从库的配置文件
   [mysqld]
   server-id=2
9. 启动从数据库
    mysqld_safe --skip-slave-start &
10. 对从库做相应的设置：
    mysql> stop slave;
    change master to master_host='127.0.0.1' master-user='repl' master-passwor='123' master-log-file='mysql-bin.000001' master-log-pos=120
11. 在从库启动slave线程
    mysql> start slave;
12. 在从库执行 show slave status\G;查看从库的状态
   也可以执行show processlist\G; 查看从服务器的进程状态
13. 测试主从是否可以正确同步
```

mysql主从复制选项：

```
log-slave-updates 用来配置从服务器的更新操作是否写入binlog，默认关闭，若这个从服务器同时是其它服务器的主，搭建一个链式的复制，则应该打开
master-connect-retry 从来设置和主服务器连接丢失时，重试的时间间隔
read-only：用来限制普通用户对数据库的更新操作，确保从库的安全性 mysqld_safe -readonly&
slave-skip-errors:忽略主从复制过程中的错误
vi /etc/my.cnf
slave-skip-errors=1007,1051,1062 设置忽略的错误号  若设置成all，表示忽略所有复制过程中的错误
```

指定复制的数据库或者表

```
1. replicate-do-table 和 replicate-ignore-table
1）启动主从数据库，在主库test中创建两个数据包，rep_t1 rep_t2
   mysqld_multi --defaults-extra-file=/etc/my.cnf start 1-3
   mysql -uroot -p -P3306 -S /tmp/mysql.sock
2）关闭数据库服务器，编辑从库配置参数replicate-do-table=test.rep_t1指定test数据库中的rep_t1表被复制 replicate-ignore-table=test.rep_t2指定test库中的rep_t2表不会被复制
   vi /etc/my.cnf
   [mysqld2]
   ...
   port = 3307
   socket = /tmp/mysql2.sock
   server-id = 2
   replicate-do-table = test.rep_t1
   replicate-ignore-table=test.rep_t2

3）启动从服务器
   mysql> start slave;
4）主从服务器都更新成功后，开始更新主数据库test库中的rep_t1表和rep_t2表
  mysql> insert into rep_t1 values(888);
  mysql> insert into rep_t2 values(888);
5）登录从数据库，检查test库中rep_t1表和rep_t2表的数据更新情况
   mysql -uroot -P 3307 -S /tmp/mysql2.sock
   检测数据库的更新情况：
   mysql> select * from rep_t1;
   mysql> select * from rep_t2;

2. replicate-do-db和replicate-ignore-db的用法
1）启动主从数据库，查询主数据库中有哪些数据库
   mysqld_multi --defaults-extra-file=/etc/my.cnf  start 1-3
   mysql -uroot -P 3306 -S /tmp/mysql.sock
   登录数据库，查看 mysql> show databases;
2）mysqldump -uroot -P 3306 -S /tmp/mysql.sock --all-databases > all.sql
3）登录从数据库，导入all.sql中的数据，保持从服务器与主服务器数据一致
   mysql -uroot -P 3307 -S /tmp/mysql2.sock
   mysql> show databases;
   mysql> source ./all.sql
   mysql> show databases; 可以看到数据库已被导入
4）关闭从库，
   mysqld_multi --dedaults-extra-file=/etc/my.cnf stop 1-3
   配置/etc/my.cnf
   [mysqld2]
   ..
   port = 3307
   socket = /tmp/mysql2.sock
   server-id = 2
   #replicate-do-table = test.rep_t1
   #replicate-ignore-table=test.rep_t2
   replicate-do-db=test
   replicate-do-db=cc
   replicate-ignore-db=tt
5）然后启动主从数据库，在主库增加cc_t1表，在tt库增加tt_t1表
   mysqld_multi --dedaults-extra-file=/etc/my.cnf start 1-3
   mysql> use cc;
   mysql> create table cc_t1(data int);
   mysql> use tt;
   mysql> create table tt_t1(data int);
6）登录从库，检查数据库是否更新
   mysql -uroot -P 3307 -S /tmp/mysql2.sock
   mysql> use cc;
   mysql> show tables;
   mysql> use tt;
   mysql> show tables;
```


查看slave的复制进度：

```
用户可以通过show processlist列表中的Slave_SQL_Running线程中的Time值得到，它记录了从服务器当前执行的SQL时间戳和系统时间之间的差距，可以得知从服务器复制的进度，从而判断从服务器上数据的完整性

1）在主服务器上插入一个保护当前时间戳的记录
   mysql> alter table rep_t3 add column createtime datetime;
   mysql> insert into rep_t3 values(1, now());
   mysql> select * from rep_t3;
2）让从服务器的IO线程停下来，是的从实践看服务器暂时不写中继日志，停止时执行的SQL就是最后执行的SQL
   mysql> stop slave;
   mysql> select * from rep_t3;
   mysql> select now();
3）从服务器上执行show processlist\G;查看SQL线程的时间，这个时间说明了主服务器最后执行的更新操作大概是主服务器46s前的更新操作
  mysql> stop slvae io_thread;
  mysql> show processlist\G;
```

mysql日常维护：

```
1. 了解服务器的状态 
   mysql> show slave status\G; 检查从库状态
   查看slave_io_running yes表明进程可以从master正确读取binlog到从服务器
   slave_sql_running yes表明进程可以读取relay-log中的SQL并解析执行
2. 服务器复制出错的原因：
   某些情况下更新会失败，首先要查看是不是主从数据库表结构不一致
   1）出现"log event entry execeeded max_allowed_pack" 错误
     可能是因为应用中使用大的blog列或者长字符串，含有大文本无法通过网络传输而导致错误
     解决： 在从库添加max-_allowed_packet参数，默认1M,set @@global.max_allowed_packet=16777216
   2）多主复制时的自增长变量冲突问题
     大多数情况下是主从复制，一对一或者一对多，单可能存在主主复制，使用auto_increment时应采取特殊步骤防止键值冲突，否则插入行时多个主库会试图使用相同的auto_increment的值
    服务器变量auto_increment_increment和auto_increment_offset可以协调多主复制和auto_increment列
    可以设置A库auto_increment_increment=1 auto_increment_offset=1
            B库auto_increment_increment=1 auto_increment_offset=0
    auto_increment_increment是每次递增的值，auto_increment_offset是每次增加后的偏移量

```

mysql主从切换：

```
假设主库A故障，需要将从库B切为主库，同时修改C的配置使其指向新的主库B
1）确保所有的从库都执行科relay log中的全部更新，看从库的状态是否是has read all relay log，是否全部更新完成
  mysql> stop slave io_thread;
  mysql> show processlist\G;
2）在从库B上停止slave服务，然后执行reset master重置成主库
  mysql> stop slave;
  mysql> reset master;
  此时发现报错 Binlog closed, can't reset master 从库B未开启binlog不能切换为主库，可以修改配置，添加log-bin选项
  [mysqld2]
  log-bin=/usr/local/var/mysql2/mysql-bin
3）配置完成后，重启数据库服务，登录B，开启主库功能
   mysql> stop slave;
   mysql> reset master;
4）在从库B添加具有执行replication权限的用户rep1，查询主数据库状态
   mysql> grant replication slave on "*"."*" to 'rep1'@'localhost' identified by '123';
   mysql> show master status;
5）在从库C上配置复制的参数
   mysql> change master to master_host='127.0.0.1', master_user='rep1', master_password='123', naster_port=3307, master_log_file='mysql-bin.0000002', master_log_pos=98;
  mysql> start slave;
6）在从库执行show slave status查看从库是否启动成功
7）在主库B和从库C上测试数据库是否可以成功同步，首先查看B库中test库中表
   mysql> use test;
   mysql> show tables;
   查看C库test库表情况  show tables;
8）在主库增加表rep_t3 
   create table rep_t3(data int);
   切换到C库，查看是否新增了rep_t3表
```

