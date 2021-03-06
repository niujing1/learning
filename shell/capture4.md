### 函数

函数的调用必须在定义之后，否则会当成一个shell命令来执行
return 只能返回0-255之间的整数值

str="1212x"
echo ${#str} 会得到字符串的长度

`$#` 输入参数的个数
`shift` 移动位置参数
`getopts` 接收函数参数

var=name
name=join
若直接输出${var}为name，输出${!var}则为join称为间接参数传递

通过全局变量传递参数
传递数组参数 func \"${a[\*]}\" //\*是当成一个参数传入 @是当成单独参数传入

0 标准输入
1 标准输出
2 标准错误输出
`>&` 用于复制一个文件描述符


数组：

```
//声明数组
array[1]=one
array[2]=two //单个赋值

array=(1,2,3) //直接赋值

//定义数组
declare -a array
arr=([one]=1 [two]=2 [4]="one") //用array([index]=val)定义

arr[index]=value //改变数组的值

//切片
${arr[@|\*:start:lenth]} //从第start下标开始，截取len个长度的数组元素，得到的是字符串，并非数组
(${arr[@|\*:start:lenth]}) //得到的是数组

//数组元素替换
a=(${arr[@]/3/100}) //将数组中的3替换成100, 替换全部

//删除数组 unset
unset arr[3] //删除数组下标为3的元素
unset arr //删除整个数组

//复制数组
arr2=("${arr[@]}")

//连接数组
join=("${arr1[@]}" "${arr2[@]}")
```


正则:

```
^ 匹配开头
$ 匹配结尾
. 匹配任意单字符
* 匹配前边的字符出现多少次
[abc] 字符集匹配，匹配任意一个即可
[0-9] 匹配0-9
[^abc] 字符集不包含a/b/c


\d 匹配数字
\D 匹配非数字
\s 空白
\S 非空白
```

其它：

```
grep -c 只显示符合要求的行数，不打印匹配信息
echo -n 不换行
fmt -w(宽度) -c(保留每个段落的缩进) -s(只折断超出宽度的行，不合并不够的行) file
rev 翻转字符串顺序
pr 格式化文本页
sort 排序 -c测试文件是否已排序
-k c1[,c2] 指定排序列,不指定c2表示一直到最后的列
-o 排序结果写入文件
-u 删除重复行，只保留第一个
-r 逆序排列
-n 按数字列自然排序
-t 指定分隔符
```

wc文本统计

```
-c 统计文本的字节数
-m 统计字符数
-l 统计行数
-L 统计最长行的长度
-w 统计单词数
```

cut 选取文本列

```
-b 只选择指定字节
-c 只选择指定字符
-d 自定义分隔符、默认制表符
-f 只选择列表中指定的文本列
-n 取消多字节字符
-s 不输出不包含列分隔符的行
cut -d ":" -f 1,6 /etc/passwd 用:做分隔符，输出其中的第1和6列
cut -d ":" -f 1-6 /etc/passwd 输出其中1-6列
cut -s -d ":" -f 1 /etc/passwd -s可以忽略到不包含正确分隔符的行
```

paste 合并文本列

```
-d 指定合并结果中的分隔符
-s 将多个文件串行的拼接在一起，即后边的文件内容追加到前一个的后边

paste和cut经常联合使用
```

join连接文本列

```
-1 filed 根据第一个文件的指定列进行联接
-2 field 根据第二个文件的指定列连接
-a filenum 是否输出不匹配的行、取值可以是1或者2 分别代表第1 2个文件
-e sring 使用参数string指定的字符串代替空列
-i 在进行关键字比较时忽略大小写
-o 自定义输出列
-t 自定义列分隔符
-v filenum 输出filenum指定的文件的所有行
类似sql的join操作，按列进行关联
eg join -1 2 -2 3 -o 1.1 1.2 2.3 a.txt b.txt 表示用a.txt的第二列和b.txt的第三列进行关联，输出a.txt的第一、第二列b.txt的第三列
```


tr 替换文件内容

```
tr [option] set1 [set2] 
-c 用字符集set2替换set1中没有包含的字符
-d 删除字符集set1中的所有字符 cat a.txt | tr -d [:digit:] 删除所有的数字
-s 压缩set1中重复的字符 cat a.txt | tr -s "[a-z]"
-t 将set1用set2替换
tr中的字符集：
[a-z] 所有的小写字母
[A-Z] 所有的大写字母
[0-9] 单个数字
/octal 一个三位的八进制数，对应有效的ASCII字符
[char\*n] 表示字符char重复出现指定次数n
[:alnum:] 所有的字符和数字
[:alpha:] 所有的字母
[:blank:] 所有水平空格
[:cntrl:] 所有的控制字符
[:digit:] 所有的数字
[:graph:] 所有可打印字符，不包含空格
[:print:] 包含空格，同上
[:lower:] 所有的小写字母
[:upper:] 所有大写字母
[:xdigit:] 所有十六进制字符
[:space:] 所有水平与垂直空格符
[:punct:] 所有标点字符

```
