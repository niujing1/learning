### debug

1. 脚本运行命令行提示错误
2. 使用`echo` 输出检测
3. 使用`trap`信号检测


```
shell 执行时会产生3个伪信号、exit、err、debug
正常结束产生exit
出错产生 err
debug在脚本的每一条命令前触发
之所以称为伪信号，是因为它是shell产生的，其它的信号都是操作系统产生的
```

例如：

```shell
#!/bin/bash

#定义信号处理函数
ERRTRAP()
{
        echo "[line: $1] Error: Command or Function exited with status code $?" 
}

#定义函数
func()
{
        return 1 #返回值设置为1 
}

#使用trap命令捕获ERR信号
trap 'ERRTRAP $LINENO' ERR #使用trap发送ERR信号到ERRTRAP，并传入参数$LINENO

#调用错误的命令
abc

#调用函数
func
```

4. 使用tee调试管道输出

`tee`可以从标准输入读取数据，然后输出到标准输出，又可以保存到文件

```
ls -lah | tee list.txt | awk '{print toupper($9)}'
```

5. 使用钩子调试脚本

```
if [ "$DEBUG" = "true" ]; then
	调试信息
fi
```

```
#!/bin/bash

#定义调试开关
export DEBUG=true

#调试函数
DEBUG()
{
        if [ "$DEBUG" = "true" ]; then
                $@
        fi
}

a=1

#调用调试函数
DEBUG echo "a=$a"

if [ "$a" -eq 1 ]
then
        b=2
else
        b=1
fi

DEBUG echo "b=$b"
```
