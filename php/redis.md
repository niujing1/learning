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
