### 文件操作

1. ls -lah 的结果10列
2. 第一列文件类型 (234)(567)(890) 3列一组、分别表示user、group、others
   每组的权限7=r(4)+w(2)+x(1)


```
. 开头文件是隐藏文件
l 是链接文件
d 文件夹
- 普通文件
rw- 代表不具备执行权限，对应类比

chmod 更改文件权限  
eg chmod u+x file 给文件添加可执行权限
chown user:group file 更改文件的所有者和所属用户组
```


find path test action 

```
常用的test条件
-name pattern 指定匹配模式的文件名
-iname pattern 指定匹配模式的文件名，不区分大小写
-type f|d 指定文件类型
-perm mode 指定文件权限位mode的文件
-user uid 查找属于指定用户的文件
-group gid 查找所有者的主组为指定组id的
-size size匹配大小为size的文件
-empty 匹配空文件
-amin [+-]n 最后一次访问时间+n n分钟之前 -n n分钟之后
-atime [+-]n 最后一次访问实际n代表天
-ctime 、-cmin、-mtime、-mmin分别是最后一次改变、修改的时间

find / -name httpd.conf 2> /dev/null 将错误信息丢弃
```


`!`条件取反

action：

```
-print 默认，写到屏幕
-fprint file 与print类似，结果写到文件中
-ls 以长格式显示搜索结果
-fls file 同ls，将结果写入文件
-delete 将搜索到的文件删除
-exec command {} \; 查找并执行命令，{}表示搜索到的文件名
-ok command {} \; 查找并执行命令，需要用户确认
\; 表示执行结束 
```

文件比较

```
comm [option] ... file1 file2
-1 不显示第一个文件中独有的文本行
-2 不显示第二个文件中独有的文本行
-3 不显示两者共有的文本行
--check-order 检测是否已排序
--nocheck-order 不检查文件是否已排序
```

diff

```
diff [option] ... file1 file2
-c 包含上下文语境的格式
-u 以统一格式显示
-y 并列方式
```


重定向：
```
> 覆盖
>> 追加
&> 代表标准输出和标准错误
> file 清空file内容或者是新建file
:> file同上  :表示一个空输出
{ date; who; } > file 将标准输出和错误都输出到file

< << 输入重定向
cat << eof eof是分隔符，生成当前文档
1&>2 标准输出被重定向到标准错误，1是2文件描述符的副本
2&>1 标准错误被重定向到标准输出，2是1 的副本
```

exec分配文件描述符

```
exec 2> file 将命令的标准错误重定向到文件
exec n< file 以只读方式打开文件，使用描述符n(n>3)
exec n<> file 以读写方式打开file，使用描述符n(n>3)
exec n>&- 关闭文件描述符n
exec n>&m 复制文件描述符m到n
```

/etc/passwd字段解释

```
登录名:x:用户id:组id:备注信息:用户主目录:默认shell程序
如果用户不需要登录，shell可以给成/sbin/nologin
用户执行shell时，父shell会根据 #! 之后的解释器创建一个子shell来执行脚本，shell执行完毕，子shell结束，回到父shell中

所谓子shell，实际上就是父shell的一个子进程
```

bash中常用的内部命令：

```
.      读取shell脚本，在当前shell中执行
alias  设置命令别名
bg     将任务放于后台执行
cd     改变当前工作目录
echo   打印当前文本
eval   将参数作为shell命令执行
exec   以特定程序取代shell或者改变当前shell的输出输入
exit   退出shell
export 将变量声明为环境变量
fc     与命令历史一起运行
fg     将作业置于前台运行
getopts  处理命令行选项
history  显示命令历史
jobs     显示后台运行的作业
kill     向进程发送信号
logout   从shell中注销
pwd      显示当前的工作目录
set      设置当前的环境变量
shift    变换命令行参数
```

保留字

```
! 逻辑非
: 空命令
break、case、continue、declare、do、done、elif、else、esac、for、
let 执行数学运算
local 定义局部变量
read 从标准输入读取一行
return、then(if)、until、wait(等待后台作业返回)、while
```

pidof 查找父进程id
$SHLVL 显示当前shell的层次

当所要执行的命令都放到一个`()`中执行时，会使shell在同一个子shell中执行
子shell可以访问父shell中的值

command& 将命令放到后台执行
shell子进程可以访问到父进程的变量，父进程无法访问子进程的变量
可以通过临时文件去传输

```shell
#!/bin/bash 
(
	#在子进程中定义变量
	x=500
	#将变量的值输出到临时文件
	echo $x > tmp
)

#在父进程直接引用x的值
echo "$x"

#从临时文件读取x的值
read b < tmp
echo "$b"
```

也可以使用命名管道

```shell
#!/bin/bash 

#创建名为fifo的命名管道
if [ -f fifo ]; then
	mkfifo fifo
fi

#子shell
(
	x=500
	echo "$x" > fifo
) &

#从管道中读取数据
read y\<fifo
echo "$y"

```

作业是针对用户而言的，一个作业可以包含一个或者多个进程


```
man ls | grep long | more 
```
这个作业实际上会启动3个进程

```
fg 切换到前台
command & 后台执行
bg 放到后台
jobs查看后台执行的程序
fg i 把序号为i的程序放到前台执行
```

作业标识符

```
[1]   Stoped ..
[2]+  Stopped .. 
[3]- Stopped .. 
[1][2][3]表示作业号，后边的+表示默认作业，执行fg时会把它移至前台运行
-表示下一个默认作业，+作业退出之后它变成默认作业 +-作业都只能有一个

Running正在运行
Done执行完毕
Stopped 作业被挂起


disown %i 删除第i个作业
```


常用的信号和含义：

```
SIGHUP    1   终端挂起或者进程终止
SIGINT    2   键盘中断
SIGQUIT   3   键盘的退出键被按下
SIGABRT   6   由abort发出的退出指令
SIGKILL   9   立即结束进程
```


`kill -s 9 pid` 向某个进程发送kill信号，终止运行



