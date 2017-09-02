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

linux下各种配置文件的加载顺序：

```
通常Linux下的配置文件：
/etc/profile  /etc/profile.d/*.sh  ~/.bash_profile  ~/.bashrc  /etc/bashrc

不同场景下的配置文件的加载机制不同，可以分3种情况
1. 正常用户登录进入bash环境：
   /etc/profile > /etc/profile.d/*.sh > ~/.bash_profile > ~/.bashrc > /etc/bashrc

2. 使用su切换用户
   ~/.bashrc > /etc/bashrc > /etc/profile.d/*.sh

3. 非登录情况 nologin
   ~/.bashrc > /etc/bashrc > /etc/profile.d/*.sh
