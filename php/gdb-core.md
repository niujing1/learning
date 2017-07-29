#### gdb调试core文件

1. core文件设置

```
1) 检测系统core文件的生成是否打开
   ulimit -c
   若显示为0，代表关闭
2）设置core文件的生成
   ulimit -c filesize|unlimit unlimit代表core文件大小无限制
   gdb调试时，若生成信息大于指定大小，生成的core文件会被截断，不完整
3）设置core文件的位置
    echo "1" > /proc/sys/kernel/core_uses_pid 添加进程id到core文件名
4）设置core文件的保存位置和文件名格式
   echo "/corefile/core-%e-%p-%t" > /proc/sys/kernel/core_pattern
   %p 文件名添加pid
   %u 文件名添加当前uid
   %g 添加当前gid
   %s 添加导致core的信号
   %t 添加生成的时间戳
   %h 添加主机名
   %e 添加命令名
```

2. core文件的查看

```
1) file 可以看到是core file
2) readelf -h core.20096 
3) 
系统异常时内核会生成core文件，可以使用gdb查看，找出导致程序异常的文件和行数
想要在整个系统中生效，想要编辑/root/.bash_profile
1）ulitmit -S -c unlimited用户级别的修改，运行所有用户生成core文件
2）修改/etc/security/limits.conf
* soft nofile 65534
* hard nofile 65534
* soft core   unlimited
* hard core   unlimited
*代表所有用户，可以指定某一用户
```

3. 求源：什么情况下才会产生core dump？

```
1. 内存越界
1）由于错误的使用下标，导致数组访问越界
2）搜索字符串，依靠字符串结束符判断字符串是否结束，但字符串没有正常使用结束符
3）使用strcpy、strcat、sprintf、strcmp、strcasecmp等字符串操作函数，将目标字符串写越界
   应该使用strncpy, strlcpy, strncat, strlcat, snprintf, strncmp, strncasecmp等函数防止读写越界。

2. 多线程程序使用了线程不安全的函数
4. 多线程读写的数据未加锁包含，对于多个线程同时访问的全局函数，应加锁
4 非法指针

1) 使用空指针

2) 随意使用指针转换。一个指向一段内存的指针，除非确定这段内存原先就分配为某种结构或类型，或者这种结构或类型的数组，否则不要将它转换为这种结构或类型的指针，而应该将这段内存拷贝到一个这种结构或类型中，再访问这个结构或类型。这是因为如果这段内存的开始地址不是按照这种结构或类型对齐的，那么访问它时就很容易因为bus error而core dump.

5 堆栈溢出.不要使用大的局部变量（因为局部变量都分配在栈上），这样容易造成堆栈溢出，破坏系统的栈和堆结构，导致出现莫名其妙的错误，或者网络读包时缓冲区溢出
```











