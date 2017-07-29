####linux的一些使用

1. 检查所有的开机启动项：

```
chkconfig --list
```

2. 设置开机启动

```
1. 编辑文件/etc/rc.local 添加需要的开机启动项
   eg、/etc/init.d/tengine start #设置tengine开机启动

2. 自己添加的shekk脚本
   放到 /etc/profile.d/文件夹下，系统启动后会自动执行

3. 通过chkconfig命令
   启动脚本放到/etc/init.d/ 或者 /etc/rc.d/init.d/下(前者是后者的软链)
   然后执行 chkconfig on
更多chkconfig的使用、请参考 chkconfig --help
```
