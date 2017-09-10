#### nginx 中logrotate日志管理工具详解

##### 简介：

```
    logrotate是一个日志管理程序、用来删除旧的日志文件、并创建新的日志文件，可以根据日志的大小或者天数来转存

	它的执行由cron服务实现、在/etc/cron.daily目录中，有logrotate、它实际上是shell脚本，用来在指定时间启动rotate，所以、使用ps无法查看到logrotate进程

    它的运行分为3步：判断系统的日志文件、建立转储计划及参数、通过fron deamon运行logrotate
启动脚本放在 /etc/cron.daily/logrotate中

#!/bin/bash

test -x /usr/sbin/logrotate || exit 0
/usr/sbin/logrotate /etc/logrotate.conf

可以人工执行测试
/usr/sbin/logrotate -f /etc/logrotate.conf
```

##### 相关参数解释

```
weekly             #每周执行一次
rotate 4           #保留4个日志文件
create             #rotate后，创建一个新的空文件
compress           #使用压缩、默认不使用
include  /etc/logrotate.d #使得这个目录下的配置文件生效

/var/log/wtmp {    #定义这个日志文件
	monthly        #每月轮转一次，覆盖上边的定义
    minsize 1M     #日志大于1M才会轮转
    create 0644 root utmp #定义新的日志文件的权限、属主
    rotate 1       #保留1个文件
    delaycompress  #延迟压缩和compress一起使用、转储的日志文件到下一次转储时才压缩
    nodelaycompress #转储同时压缩
    copytruncate   #用于还在打开中的日志文件、把当前日志文件备份并截断
    nocpoytruncate #备份日志文件、但不截断
    ifempty        #即使是空文件也转储
    errors address #转储时的错误信息发送到指定的email地址
    nomail         #转储时不发生日志文件
    olddir dir     #转储后的日志文件放入到指定目录、必须和当前日志文件在同一个文件系统
    noolddir       #转储后的文件和当前日志文件放到同一个目录
    prerotate/endscript #转储之前需要执行的命令可以放入、这两个关键字必须单独成行
    postrotate/endscript #转储之后需要执行的命令可以放入、这两个关键字必须单独成行
    size size      #日志大于指定大小时才转储
    sharescript    #共享脚本、下边的postrotate<s>endscript只执行一次即可
}
```


