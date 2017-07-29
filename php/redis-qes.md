### 一次redis问题排查

由一次applog引发的redis问题排查：

```
发现调用第三方的数据虽然使用了redis来缓存，接口响应依然缓慢，查了下日志，发现:
[error] 944#0: *274 FastCGI sent in stderr: "PHP message: PHP Notice:  Redis::get(): send of 43 bytes failed with errno=32 Broken pipe
```

因为咱们redis的连接使用的是pconnect，第一反应就是pconnect会有问题....

一番google、发现、好多说pcntl不能放到父进程去处理、必须放到子进程~~~
搜了搜咱们项目也没用到pcntl这种进程的处理，请教了下朋友，他提到了几个方面,下边会一并梳理处理：go on~


TIPS:
```
安装完redis一定要修改配置文件：
查找filelog、添加错误日志文件
vi /usr/local/matrix/build/redis/redis.conf #这是我自己的redis配置文件的位置，可以对应查找自己的配置文件

修改为：logfile "/usr/local/var/log/redis.log"

这样、下边的排查才能继续
```


1. redis启动过程的警告
```
1). 警告(WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.)解决办法
   方法1: 临时设置生效: sysctl -w net.core.somaxconn = 1024
   方法2: 永久生效: 修改/etc/sysctl.conf文件，增加一行
          net.core.somaxconn= 1024
   然后执行命令
   sysctl -p

2). (error) MISCONF Redis is configured to save RDB snapshots, but is currently not able to persist on disk. Commands that may modify the data set are disabled. Please check Redis logs for details about the error. 

    运行config set stop-writes-on-bgsave-error no 
    或者修改sysctl vm.overcommit_memory=1

3) WARNING you have Transparent Huge Pages (THP) support enabled in your kernel. This will create latency and memory usage issues with Redis. To fix this issue run the command 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' as root, and add it to your /etc/rc.local in order to retain the setting after a reboot. Redis must be restarted after THP is disabled.
  按照提示、终端输入 echo never > /sys/kernel/mm/transparent_hugepage/enabled(root权限下，非root用户登录要加sudo临时提权)
  然后编辑/etc/rc.local 
  vi /etc/rc.local
  追加 echo never > /sys/kernel/mm/transparent_hugepage/enabled
  
检查完这几项之后，重新启动redis、可以发现redis启动日志中的warning和error信息不存在这几项提示了
```

2. 连接失败，可能原因：
```
1）检查redis-server是否启动  ps axu | grep redis

2) 测试开发机本机是否可以连接成功、redis-cli 
   可以使用ping测试、响应pong信息、则表示可以正常

3）测试开发机phpredis的ping是否正常响应

4）检查本地redis扩展是否正常？ php -m | grep redis

5) 检测本地redis扩展的版本  php --ri redis
   有扩展版本过低的情况也可能会导致一些错误

6）本地连接开发机测试redis connect是否成功？
   try{
     $redis = new Redis();
     $con = $redis->connect('10.30.128.242', '6379');
	 var_export($con);exit(); //查看是否连接成功
   }catch(Exception $e){
      var_export($e);
   }
   
7）查看本地php连接开发机redis的响应
   try{
     $redis = new Redis();
     $con = $redis->connect('10.30.128.242', '6379');
     // var_export($con);exit(); //查看是否连接成功
     $res = $redis->ping();
     var_export($res);//检测服务器响应消息是否为pong
   }catch(Exception $e){
      var_export($e);
   }
如果、如果、你遇到了和我一样的问题、con成功、ping响应失败，导致下边一系列的redis的操作都是失败的，请继续往下看~~~

1）登录开发机
   检查防火墙状态 service iptables status
   修改配置文件 vi /etc/sysconfig/iptables
   添加 -A INPUT -p tcp -m tcp --dport 6379 -j ACCEPT
   关于iptables的规则比较麻烦、有兴趣可以参考：
   http://blog.csdn.net/reyleon/article/details/12976341
 
2) redis.conf文件里的bind配置，只监听了localhost,没有监听0.0.0.0(公网)
   注释掉 bind 127.0.0.1那行

3) Redis is running in protected mode because protected mode is enabled
   修改config文件、protected-mode设置为no
   重启redis

4）测试redis操作的状态、是不是可以正常的响应
   如果可以的话，就基本上没问题的

5）get、set失败不是必现，是偶发
   a. 检查网络状况，是不是网络原因导致redis连接不稳定
      可以在set或者get前ping一次，若失败，重连一次redis
   b. 使用pconnect的时候，注意一个问题 pconnect是不释放连接的，直到php进程结束掉，在高并发的情况下，有可能会导致文件描述符不够用
   c. 可以捕获下redis set失败时的异常信息、根据信息作出具体的判断
 
```


redis出现问题的原因大概就以上这些

#### redis的一些使用：

1. 启动

```
redis-server 使用默认配置文件启动
redis-server /path/redis.conf 使用指定配置文件启动
redis-server --port 6390 让redis在指定端口启动
redis-server --port 6390 & 让redis在指定端口以deamon形式运行

redis-cli 启动默认端口的cli
redis-cli -p 6390 启动指定端口的redis-cli
redis-cli -h 10.30.128.242 -p 6379 连接指定主机的redis
```

2. redis配置项说明

```
bind 192.168.1.100 指定可以接收来自哪个主机的连接
默认bind 127.0.0.1 就是不允许远程连接
protected-mode no  是否运行在保护模式、yes的话，不允许远程连接
port  监听的端口
unixsocket /tmp/redis.sock 使用socket连接时，sock文件
timeout 0 连接超时时间、0为不限制
tcp-keepalive 300 
daemonize no 是否以deamon形式运行、否认否
pidfile /var/run/redis_6379.pid 进程id保存的文件
loglevel notice 错误日志的级别
logfile "/usr/local/var/log/redis.log" 错误日志
databases 16 默认数据库的个数
save 300 10 如果在300s内发生的改变超过10次，就写入一次日志文件
rdbcompression yes rdb文件是否启用压缩
dbfilename dump.rdb 日志文件的名字
slaveof <masterip> <masterport> 配置主从redis
repl-* 这一类的都是和redis复制相关的
slave-priority 100 配置从redis的优先级
appendonly no 是否开启aof
appendfilename "appendonly.aof" aof的文件名
appendfsync everysec aof的写入机制
```
