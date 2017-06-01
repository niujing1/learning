### 系统性能监控

ps：

```
ps -ef 查看所有进程
kill -9 pid 终止某个进程
```

top

```
tasks表当前作业相关
total 进程总量  running正在运行的进程数
sleeping 表示睡眠的进程数量
stopped 停止的
zombie 僵尸进程
us用户空间占用CPU的百分比
sy内核空间占用CPU的百分比
ni用户进程空间内改变过优先级的进程占用CPU的百分比
id空闲CPU的百分比
wa等待输入输出的CPU的百分比
```

vmstat

```
r 表示可运行进程的数量
b 表示阻塞进程的数量
swpd 已经使用的交换空间的数量
buff 缓存使用的ram的数量
cache 文件系统缓存使用的ram的数量
free 自由ram数量

si 磁盘分页到内存的数量
so 内存分页到磁盘的数量

bi 从磁盘读入的块
bo 写入磁盘的块

in 系统中断
cs 进程上下文开关
us 用户模式
sy 内核模式
wa 等待io
id 空闲状态
```

`mytop` 类似top风格的MySQL监控工具

iostat

```
iostat [opt]
-c 仅显示CPU的状态
-d 仅显示存储设备的状态，不能和-c一起使用
-k 默认显示的是读入读出的块信息
-t 显示搜集数据的时间
-V 显示版本号和帮助信息
-x 显示扩展信息

结果解释：
%user 在用户级别运行所使用的CPU的百分比
%nice nice操作所使用的CPU的百分比
%system 系统级别运行所占用的CPU的百分比
%iowait CPU等待硬件IO时，所占用CPU的百分比
%idle CPU空闲时间所占用CPU的百分比

Device 设备块的名称
tps 每秒钟发送到的IO请求数
kB_reads/s 每秒从该设备读取的数据块数量
Blk_wrtn/s 每秒从该设备写入的数据块数量
Blk_read 从该设备块读取的数据块的总和
Blk_wrtn 写入总和
```

sar或者开源Nagios、cacti

