#### 使用strace跟踪进程调用

常用选项：

```
-c 统计每次系统调用执行的时间、次数和出错的次数等
-d 输出strace关于标准错误的调试信息
-f 跟踪由fork产生的子进程
-ff 若提供-o filename，则所有的跟踪结果输出到相应的文件中，pid是进程名
-F 尝试跟踪vfork调用
-i 输出系统调用的入口指针
-t 在输出结果的每一行加上时间信息
-tt 同上，微妙级别
-T 显示每次调用消耗的时间
-v 显示所有的系统调用、包括环境变量、输出输入等
-V 显示strace的版本信息
-e expr指定表达式控制如何跟踪
[qualfier=]![value][,value2]...
qualfier只能是trace、abbrev、verbose、raw、signal、read、write之一
-eopen等价于 -e trace=open表示只跟踪open调用
-e trace=file 只跟踪有关文件操作的系统调用.
-e trace=process 只跟踪有关进程控制的系统调用.
-e trace=network 跟踪与网络有关的所有系统调用
-e strace=signal 跟踪所有与系统信号有关的 系统调用
-e trace=ipc 跟踪所有与进程通讯有关的系统调用
-p pid 跟踪指定的进程id
```

2. 使用示例

```
1. 跟踪可执行程序
   strace -f -F -o ~/straceout.txt vim 跟踪vim的进程调用
2. strace -o output.txt -T -tt -e trace=all -p 28979 跟踪进程
```

3. lsof

```
1. 列出指定协议
   lsof -i 4 列出ipv4协议的进程
2. lsof -p pid 列出某个进程的子进程
3. lsof -i 列出指定端口
4. lsof -i tcp 列出指定协议
```

4. tcpdump查看数据包

```
tcpdump -i eth0 最简单、但是刷的太快，基本上看不到
tcpdump -i eth0 -nn 'host 10.33.4.37' 监听指定主机的包
tcpdump -i eth0 -nnA 'port 80' 监听指定端口
tcpdump -i eth0 -nnA '!port 22' 监听所有非22端口的数据包
```

