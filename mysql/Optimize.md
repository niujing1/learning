### MySQL性能优化

查询数据库的一些性能

```
show status like 'value';

常用的value：
connections：连接mysql服务器的次数
uptime：服务器上线时间
slow_queries：慢查询的次数
com_insert/update/select/delete 对应操作的次数
```

SQL优化的思路：

```
1. 优化更需要优化的查询，eg 一个查询1w次/min，每次30次IO，另一个100次/min，3000次IO，假设优化第一个查询，从30个IO -> 25会很容易，第二个查询想要达到相同的效果，需要降低500IO，显然第一个优化要容易

2. 定位优化对象，可以使用show profiles来观察是哪里耗费了太多的性能

3. 明确的优化目标

4. 充分利用explain和profile来分析语句
```

explain结果分析：

```
1. id  select识别符，就是select查询序列号，id的值越高，优先级越高，越先被执行
2. select_type：可能的取值如下：
   1）simple表示简单查询，不包括子查询和连接查询
   2）primary表示主查询或者最外层的查询
   3）union和union result表示连接的第二个或者后边的查询，不依赖于外部查询的结果集
   4）dependent union连接查询中的第二个或者后边的select取决于外边的查询
   5）subquery子查询中的第一个查询不依赖于外部结果集
   6）dependent subquery依赖
   7）derived from子句有子查询，MySQL会递归执行这些子查询
3. table：查询是关于哪张表的
4. type：连接类型
   1）system 仅有一行的系统表，是const连接的特例
   2）const最多只有一个匹配行，在余下的查询中会被当成常量对待，查询很快，因为只被读取一次
   3）eq_ref：对于每个来自前边的表的行组合，从表中读取一行，当一个索引的所有部分都在查询中使用，并且索引是unique或者primary key的时候会使用,可以用于=操作带索引的列
   4）ref：对于来自前边的表的任意行组合，将从该表读取所有匹配的行，可用于=或者<=>操作的带索引的列
   5）index_or_null：通ref但是添加了可以专门搜索NULL的行
   6）undex_merge：使用了索引合并优化，此时key_len包含了使用索引的最长的关键元素
   7）range：只检索指定行
   8）index：同all，单比all快，因为索引文件一般比数据文件小
   9）all：完整扫描
5. possiable_keys MySQL可以通过哪个索引来找到该行

6. key查询实际用到的索引
7. key_len实际使用到的索引长度
8. rows执行查询时必须检查的行数
9. extra附加信息
   using index使用了覆盖索引，避免了访问不必要的列，效率不错
   using where 服务器收到行记录后会进行过滤
   using temporary 在对查询结果排序时用到了临时表
   using filesort 会对数据使用一个外部的索引排序

```

使用profiling分析查询语句：

```
show profile [type [, type]..] [for query n] [limit row_count [offset offset]]
type 可以为： 
all：显示所有信息
block io：显示输出输入操作阻塞的数量
cpu：显示系统和用户CPU使用的时间
ipc：显示信息发送和接收的数量
memory：内存的信息
page faults：显示主要的page faults数量
source：显示函数的名称，并且显示函数所在文件的名字和行数
swaps：显示swap数量
```

如何使用索引提高查询速度：

```
1. 使用like关键字的查询，%不在第一个位置时，索引才会生效
2. 使用多列索引的查询语句，只有当使用了第一列的时候，索引才会生效
3. 使用or关键字的查询，or前后的两个条件都是索引列时才会被使用
```

不同类型的SQL优化方法：

```
1. 优化insert语句
   insert执行的大概步骤为：
   1）客户端连接MySQL服务器
   2）客户端发送insert语句到服务器
   3）服务器解析insert数据
   4）服务器增加数据
   5）服务器给增加的数据添加索引
   6）服务器关闭连接
  a. 需要插入大量记录到MySQL时，可以尽量选择一次性插入，减少S-C连接和关闭的时间，并且当从一个文本文件装载一个表的时候，使用load data infile加载数据往往比使用很多insert语句效率要高20倍
  b. 对于MyISAM类型的表，若从不同客户端插入很多行，可以使用insert delayed语句提高执行效率，insert delayed into是客户端将数据提交给Server，但插入并不立即执行，而是返回一个成功的状态给client，等待服务器空闲的时候执行写入，此时数据并未真正的写入到磁盘，好处是可以提高插入的效率，坏处是若系统崩溃，还未写入磁盘的数据将会丢失
  c. 可以锁定表加快写入的速度
2. 优化order by语句
  a. 对于 select [col] from table where colx=val order by [sort] limit [offset],[limit]类型的语句要在colx和sort上建立联合索引
  b. 对于order by + limit类型的，只在order by后的字段上加上索引即可
  c. 不要对where和order by的选项使用函数或者表达式
  d. 不应该使用索引的情况：1）order by的字段混合asc和desc
     2）where子句使用的字段和order by使用的字段不一样
     3）对不同的关键字使用order by排序
3. 优化group by
   使用group by时，MySQL会自动对符合条件的结果排序，通过扫描整个表，并创建一个临时表，表中每个组的所有行应为连续的，然后使用该临时表来找到组并应用累计函数，某些情况下可以使用索引排序，而不用临时表 指定order by null
4. 优化嵌套查询
   使用连接查询join代替子查询，省去临时表的创建和撤销的时间
5. 优化or条件 在or两边的条件上都建立索引，or是先对or的各个字段在查询结果之后在进行union操作
```

优化插入记录的速度：

MyISAM表

```
1. 禁用索引 
   插入记录时，MySQL会根据表的索引对插入的记录建立索引，若插入大量数据，建立索引会降低插入的速度，可以在插入之前禁用索引，之后再打开索引
  alter table tb disable keys、alter table tb enable keys
2. 禁用唯一性检查
   MySQL在插入时还会对记录进行唯一性校验 set unique_checks=0禁用 =1开启
3. 一次插入多条记录
4. 使用load data infile导入  load data infile `/home/mysql/data/test.txt` into table tmp;
```

InnoDB表：

```
1. 禁用唯一性检查
2. 禁用外键检查
3. 禁止自动提交 set autocommit=0
   load data infile `/home/mysql/data/test.txt` into able tmp;
   set auto commit=1
```

优化数据库结构：

```
1. 将字段很多的表分解成多个表，不常用字段分离出来
2. 增加中间表，分析经常需要联合查询的字段，使用这些字段建立一个中间表，将原来联合查询的数据插入到中间表中，这样可以直接从中间表读取数据，而不用每次联合查询
3. 增加冗余字段，减少联合查询
```

分析表、检查表和优化表

```
分析表： analyze table tb;
检查表：check table tb [option]; option可以是：
1）quick：不扫描行，只检查错误的连接
2）fast：只检查没有被正确关闭的表
3）changed：只检查上次检查后修改的表和没有被正确关闭的表
4）medium：扫描行，以验证被删除的连接是有效的
5）extended：对每行的所有关键字进行全面查找，比较耗时
优化表：optimize table，但是只能优化varchar、blob或者text类型的字段
```

注意：option只对MyISAM表起作用，对InnoDB表无效，check table在执行过程中也会给表加上只读锁


