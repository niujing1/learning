```
nginx 学习：
(一)、基本概念和安装
系统硬件：CPU >= 2core   内存 > 256M

1. 确认网络可用
2. 确认yum可用
3. 关闭iptables
4. 关闭selinux
iptables -t nat -F
iptables -t nat -L 

常见的http服务：
httpd - apache
iis - 微软
gws - google(不对外开放)

为什么选择nginx：
1. 采用IO多路复用

串行、多线程、IO多路复用(主动上报 在一个线程内交替顺序执行，这里的复用是指复用同一个线程 -- 实现方式select poll epoll)、
select 会阻塞、直到内核态发送可用请求，会一直变量所维护的文件描述符
缺点：1）可监视的最大文件描述符的数量是一定的 max：1024
     2）线性扫描效率低下
epoll模型：当fd就绪、采用系统的回调换算将fd放入、效率更高
           无最大连接数的限制

2. 轻量级：
   1）功能模块少、非主要功能模块默认不安装、需要自己指定安装
   2）代码模块化

3. cpu亲和 (affinity)
   1）cpu亲和(把工作进程和cpu核心绑定、减少切换cpu造成的cache miss(会带来一定的性能损失))
      使用sendfile(在处理静态资源时、开启sendfile、可以不用经过用户态、只经过内核态、直接传递给用户)
      file->kernel->user->user->kernel->socket
      sendfile：file->kernel->kernel->socket

yum源：
vi /etc/yum.repos.d/nginx.repo
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/mainline/OS/OSRELEASE/$basearch/
gpgcheck=0
enabled=1
   
rpm -ql nginx
/etc/logrotate.d/nginx nginx日志切割
查看nginx编译参数 nginx -V

work_process 连接进程数
woker_connections 最大连接数


nginx 调用lua模块指令
nginx的可插拔模块化加载执行，共11个处理阶段

set_by_lua
set_by_lua_file 设置nginx的变量、可实现复杂的逻辑判断

access_by_lua
access_by_lua_file  请求访问阶段处理、用于访问控制

content_by_lua
content_by_lua_file  内容处理器、接收请求处理并输出响应

nginx.var nginx变量
ngx.req.get_headers 获取请求头
ngx.req.get_uri_args 获取url请求参数
ngx.redirect 重定向
ngx.print 输出响应体
ngx.say  同ngx.print 最后会输出一个换行符
ngx.header 输出响应头


灰度发布：
1. 根据用户信息的cookie等信息区别
2. 根据用户ip区别

多个虚拟主机使用相同server_name时，优先读取最先读取的配置、没有绑定server_name的ip也是优先读取最先读取的配置

location优先级

= 精确匹配
^~ 普通字符匹配，使用前缀匹配
上边两个的优先级最高、一旦匹配到，就不再进行剩下的匹配

(区分大小写)~   (不区分)\~* 表示执行正则匹配、匹配到之后还会继续匹配、找到匹配最完整的


try_files 按顺序检测文件是否存在

location root | alias
location /request_path/img/ {
  root /test/aaa/img;
}
location /request_path/img/ {
  alias /test/aaa/img;
}

若请求 www.ngtest.cc/request_path/img/test/aaa/img/cat.png 
实际请求分别是：
www.ngtest.cc/request_path/img/test/aaa/img/cat.png
和 www.ngtest.cc/request_path/img/img/cat.png


传递用户真实ip
set_real_ip=$remote_addr

限制用户上传文件 client_max_body_size  413 request entity too large
后端服务无响应 502 bad_gate_way
后端服务执行超时  504 gateway timeout

 Linux、unix一切皆文件、文件句柄就是一个索引
 设置方式：
 设置全局级别、设置用户级别、设置进程级别
   vi /etc/security/limits.conf 
   root soft nofile num1 root用户、超过num1系统不会强制限制
   root hard nofile num2 root用户、超过num2系统会作出限制，请求会收到影响
   *    soft nofile num1 所有用户、所有进程
   *    hard nofile num2 所有用户、所有进程
   
   cpu的亲和
   把进程与处理器绑定、是的进程在处理器之间频繁迁移进程的几率减少、减小性能损耗
   nginx配置文件中 work_rlimit_nofiles 进程级别的请求限制
   cat /proc/cpuinfo | grep "physical id" | sort | uniq | wc -l 物理CPU的核数
   cat /proc/cpuinfo | grep "cpu cores" | uniq
   cat /proc/cpuinfo | grep "processor" | wc -l
   
   work_process 2； 进程数、应该和电脑的核心数一致
   work_cpu_affinity 0000 0000 0000 0001    0000 0000 0000 0010 对应绑定到cpu的核心上
  #work_cpu_affinity auto
  
  ps -eo pid,args,psr | grep nginx
  psr 就是使用的核心数
  auto是nginx1.9之后支持的、不用再一一列出，nginx会自动帮我们绑定
  events事件驱动器
  use epoll；
  work_connections 10240; 限制每个进程可以处理的连接数，总共可以处理work_connections* work_process
  work_rlimit_nofiles 65535； 每个进程的打开的最大句柄数
  
  http模块下可以设置charset 
  charset utf-8;
  tcp_nopush on; 打包发送
  gzip_disable "MSIE [1-6]\" ie6以下不支持压缩、会导致开启gzip压缩的图片无法展示、可以这样如果是ie6以下就禁用压缩
  

```

