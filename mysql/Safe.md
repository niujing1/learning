### MySQL权限与安全

user表包括用户权限、资源控制
db和host表存储用户对某个数据库的操作权限，决定用户可以从哪个主机存取哪个数据库
procs_priv可以对存储过程和存储函数设置操作权限

mysql -h -u -p -e。。
其中-e是执行SQL语句，若指定了改参数，可以在登录数据库后执行该语句
-D指定数据库

修改root密码：

```
1. 使用mysqladmin -u root -p password 'pass'按照提示修改
2. 修改user表 update mysql.user set Password=PASSWORD("oldpass") where user='root' and host='localhost'执行update后需要使用flush privileges重新加载用户权限
3. 使用set修改  set password=password("oldpass")
```

root修改普通用户的权限

```
1. 使用set修改
set passwd for 'user'@'host' = password('oldpass');
2. 使用update修改
update mysql.user set password=password('newpass') where user='uname' and host='host'
3. 使用grant修改
grant usage on "*" to 'user'@'%' identified bbby 'newpass';
```

root密码丢失：

```
1. 停止MySQL服务
2. 使用--skip-grant-tables启动mysql  Linux下最好采用mysqld_safe来启动
mysqld_safe --skip-grant-tables
3. 使用root登录重新设置密码
4. 使用flush privileges;加载权限
```




