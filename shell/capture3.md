### 条件测试和判断语句

1. 使用test判断
2. 使用 `[` 判断，它是一个命令，所以需要和用来测试的表达式之间隔一个空格
3. 变量赋值的时候，前边一定不能有空格，否则会被当成shell命令对待
4. shell区分大小写


5. 字符串运算

运算符      | 说明
------------|----------
string      | 判断指定字符串是否为空(这种情况只能使用test)
str1i = str2   | 是否相等(等号的两边要有空格，最好把str1和str2用引号包括)
str1 != str2  | 是否不相等
-n str      | 是否为非空串
-z str      | 是否是空串

6. 数字运算符

运算符      |  说明
------------|----------
num1 -eq num2 | 是否相等
num1 -ne num2 | 是否不等
num1 -gt num2 | >
num1 -lt num2 | <
num1 -ge num2 | >=
num1 -lt num2 | <=

7. 文件操作

操作符        | 说明
--------------|----------
-x            | 文件是否为可执行文件
-f            | 文件存在，且为普通文件
-u            | 是否设置了suid位
-L            | 是否存在且为链接符号
-d            | 存在且为目录
-b            | 存在且为块文件
-c            | 存在且为字符文件
-s            | 存在且非空



```
chmod u+x file 更改文件权限
chown user:group file 更改用户组
```

`!expr` 逻辑非
`expr1 -a expr2` 逻辑与
`expr1 -o expr2` 逻辑或

条件判断

`if...then...else...fi`

`read`读取终端输入

多条件判断：

```
case var in 
val1)
   sta1
   sta2;;
val2)
   sta1;;
*)
 default;;
esac


[[:lower:]] 小写字母集合
[[:upper:]] 大写字母集合
[0-9] 数字

** 幂运算

数学运算的四种方式：
let n=n+1
expr 2-100
$((2-100))
$[2-100]

使用非十进制
((x="2#100")) 
echo $x //二进制
((0x100)) //8进制
#隔开或者传统的前缀法

//循环
for var in list
do
...
done

array=(1 2 3)
for i in ${array[*]}

until expr 
do
...
done

while [[ expr ]]
do
...
done

```
