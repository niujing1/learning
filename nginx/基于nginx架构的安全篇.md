#### 基于nginx架构的安全篇

```
  1. 常见的恶意行为
     1）爬虫行为和恶意抓取、资源盗用
     2）基础防盗链功能 - 不让恶意用户轻易的爬取网站对外数据
     3）secure_link_module - 对数据安全性提高加密验证和时效性、适合如核心重要数据
     4）access_module - 对后台、部分用户服务的数据提供IP防控
  2. 常见的应用层攻击手段
  	 1）后台密码撞库 - 通过猜测密码字典不断对后台系统登录性尝试，获取后台登录密码
  	    a. 加强后台密码的复杂度
  	    b. access_module - 对后台提供ip防控
        c. 预警机制
  	 2）文件上传漏洞 - 利用文件上传接口将恶意代码植入到server，再通过url访问可执行代码
  	    upload 要单独控制、限制php文件的执行
  	    location ^~ /upload {
           root /usr/local/img;
           if ($request_filename ~* (.*)\.php) {
             return 403;
           }
  	    }
  	 3) SQL注入 - 利用未过滤/未审核用户输入的攻击方法、让应用运行本不该运行的SQL代码
  	    场景：用户 -> nginx+lua -> fastcgi -> php -> sql -> mariadb
  	     ' or 1=1 # 
         select * from users where username = '' or 1=1#' and password='123'
       当然可以在php代码层进行严格检测
       、
       也可以在nginx层进行nginx+lua的抓取和执行预计、来进行对应策略
       nginx+lua -> {拦截cookie类型攻击、拦截post请求、拦截cc攻击、拦截url、拦截arg} -> {java/php/python}
       cc攻击：就是连续不断的访问、使得后端无法真正的提供服务
       url：就是防止一部分url、不允许其访问
       arg：就是拦截请求参数
  3. Nginx防攻击策略
  4. Nginx+Lua的waf防火墙
     1) 了解需求
        定义nginx在服务体系中的角色
        a. 静态资源服务(类型分类、flv、image、js、css | 浏览器缓存设置 | 防盗链 | 流量限制 | 压缩 )
        b. 代理服务(协议类型 | 正向反向代理 | 负载均衡 | 代理缓存设置 | 头信息处理 | 分片请求 | prooxy_pass)
        c. 动静分离
        
     2）设计评估
        a. 硬件 (CPU | 内存 | 硬盘)
        b. 系统 (用户权限 | 日志目录存放)
        c. 关联服务 (LVS、keepalive、syslog、fastcgi)
      
     3）配置注意事项
     	a. 合理配置
     	b. 了解原理
     	c. 关注日志
        
```

