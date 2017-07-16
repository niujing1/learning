### 其它信息

系统启动的过程：

```
按下电源-->加电-->执行bios程序-->完成硬件自检
硬件没问题，调用内核装载器载入系统内核，比较流行的是GNU、GRUB
-->执行/sbin/init程序-->读取/etc/inittab文件，执行初始化操作
-->用户登录

init进程是系统启动后第一个进程 id=1 是所有进程的父进程
它读取/etc/inittab、/etc/init/rc.conf、/etc/init/rcS.conf等配置文件
进行初始化操作，停止或启动某些进程

/etc/rc.d/rc 的作用是根据不同的运行级别来停止或者启动服务
```

运行级别：

```
0 停机
1 单用户模式、不启用网络和各种服务，只运行root登录维护
2 多用户模式，不启用网络和各种服务
3 多用户模式、除图形界面之外的各种网络服务都可以正常使用
4 用户自定义
5 带图形界面的多用户模式
6 重启系统
```

```
其实这里的脚本是init.d里边的一个符号连接
若文件首字符是S表示start将传递/etc/init.d/下对应的脚本启动服务
若首字母是K表示stop停止服务
 