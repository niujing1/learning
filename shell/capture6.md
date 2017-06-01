### awk

####工作流程

1. 自动从指定的数据文件中读取行文本
2. 自动更新awk的内置系统变量的值，如列数变量NF、行数变量NR、行变量$0
3. 依次执行程序中的所有匹配模式及其操作
4. 执行完所有的匹配模式后，若有未读取内容，重复1-3

awk关系表达式：

```
`>` `<` `==` 
 eg awk '$2 > 80 {print}' scores.txt 输出第二列的分数大于80的记录
```

使用正则表达式

```
awk '/^T/ {print $2}' 打印以T开头的行的第二列
```


混合模式

```
awk '/^T/ && $2 > 80 {print $2}' 打印以T开头并且第二列大于80的记录的第二列
```


begin模式

```
awk程序中只包含begin时，不需要指定数据文件、通常用户可以把一些初始化操作放在begin模式，如自定义的行列分隔符、初始化变量等

是在awk程序刚开始，但又未读取数据之前，对应的操作仅被执行一次，awk读取数据之后，begin模式不在成立，可以把只执行一次，且与数据文件无关的操作放到begin中
```

end模式

```
与begin模式相反，是awk处理完所有的命令，即将退出程序时，可以将善后工作放到end中
```

变量定义和引用

```
定义 x=3 输出 print x
```

awk内置变量

```
$0 记录变量，表示正在处理的记录
$n 字段变量、其中n为整数，表示第n个字段的值
NF 整数值，表示当前记录的字段数
NR 整数值，表示awk已读入的行数
FILENAM 表示正在处理的数据文件的每次
FS 字段分隔符，默认空格或者制表符
NS 记录分割符，默认换行
RSTART 表示正则表达式在父串中出现的位置
RLENGTH 表示匹配长度
```

其它运算

```
条件运算 $2>90 ? "xxx" : "xxx"

逻辑运算
&& || ! 

关系运算
> < >= <= == != ~ !~
```

函数：

```
index(str1, str2) //返回str2 在 str1中的位置，若多次出现，返回第一次出现的位置，若不包含，返回0
length(string) //返回字符串的长度
match(str, regexp) //在str中搜索符合正则表达式regexp的子串，若匹配多个，返回第一个
split(str, array, seperator) //根据sep将str分割成数组，存放在array中
substr(str, start, [length]) //从str截取指定子串，不指定length则截取到最后
sub(regexp, repalcement, string) //将str中符合reg的第一个子串替换为repalce
gsub(regexp, repalcement, string) //替换全部符合的子串

算数函数
int(x) 返回x的整数部分
sqrt(x) 返回x的平方根
exp(x) 返回e的x次方
log(x) 以e为底的对数值
sin(x) 正玄值
cos(x) 余弦值
rand() 0-1之间的随机数
srand([x]) 以x为种子返回一个随机数
```


数组

```
awk数组名区分大小写，下标从1开始
直接赋值即可 for(n in arr)
```

流程控制

```
if()..else...
while
do..while
for 
break
next与continue类似，但可以用在整个awk程序中
print、printf、sprintf后两个用于格式化输出，sprintf不会输出到屏幕
```
