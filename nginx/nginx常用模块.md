#### nginx常用模块

```
1. stub_status
   应用： http、server段
    location /ngx-status {
        allow 127.0.0.1;
        deny all;
        stub_status on;
        access_log on;
    }
    
2. http_random_index_module 从目录中随机选择一个主页
   random_index on|off;
   应用：location
   
3. http_sub_module http内容替换
   sub_filter string replacement
   sub_filter_last_modified on | off; ->cache
   sub_filter_once on | off; 返回匹配的第一个还是全部
   应用：http、server、location段
  
4. limit_conn_module 连接限制
   limit_req_module 请求频率限制
   http请求是建立在tcp连接之上，一次连接可以处理多个http请求
   limit_conn_zone key zone=name:size;(name是空间名称 size是空间大小)
   limit_conn zone number; 限制多少个这样的空间大小
   应用：http、server、location段
   
   limit_req_zone key zone=name:size rate=rate;(name是空间名称 size是空间大小)
   limit_req_zone zone [burst=number] | [nodelay]; (burst放到下次处理的请求数、nodelay直接拒绝)
   http {
    limit_req_zone $binary_remote_addr zone=one:10m rate=1r/s;
    ...
    server {
        ...
        location /search/ {
            limit_req zone=one burst=5;
        }
    }
   
5. 访问控制：
   http_access_module 基于ip
   http_auth_basic_module基于用户认证
   location / {
    deny  192.168.1.1;
    allow 192.168.1.0/24;
    allow 10.1.1.0/16;
    allow 2001:0db8::/32;
    deny  all;
 }
 应用：http、server、location段
 
 6. http_x_forwarded_for包含所有经过的代理的ip
    解决经过代理后无法获取用户真实ip
    1). 采用http头信息控制访问 http_x_forwarded_for
    2). 结合geo模块操作
    3). 使用http自定义变量
  
7. nginx的使用场景
   1). 静态资源web服务器
   2). 代理服务器
   3). 负载均衡调度器
   4). 动态缓存
 
8. 在send_file开启的情况下、一般配合tcp_nopush提搞网络包的传输效率
   在keep_alive开启的情况下、一般配合tcp_nodelay提高数据包传输的及时性
   
9. http缓存机制
   brower ->  cache -> expire ? N -> display
                                Y -> etag    ->   if_none_match
                                      |               |
                                last_modify  ->    if_modified_since
                                      |
                                 web server  ->    200 / 304
                                      |                |
                                     resp             304
                                                       |
                                                     read cache
                               display       <-        |
                               
10. 防盗链：
    要区别哪些是正常的用户请求，基于http_refer
    valid refer none blocked ip;
    eg . valid_refer none blocked 10.30.11.23;
        if ($valid_refer) {
          return 403;
        }
    所有非来自10.30.11.23的请求都将设置valid_refer=1,在下边会被直接return 403；
    
 11. nginx作为代理
     正向代理是代理客户端，为客户端收发请求，使真实客户端对服务器不可见；
     而反向代理是代理服务器端，为服务器收发请求，使真实服务器对客户端不可见。
     eg. 正向代理一般用作局域网访问internet、多个局域网电脑使用同一个出口
         反向代理一般多个client使用多个server、具体访问到哪一个server对于client是不透明的、随机的
  
 12. proxy_buffering on | off; 是否开启代理缓冲
     proxy_Redirect  default | off | repleacement; 跳转重定向
     proxy_set_header filed value；设置头信息
     proxy_hide_header; 隐藏代理头信息
     proxy_Set_body; 设置代理请求体
  
 13. SLB分为4层负载均衡和7层负载均衡
     4层负载均衡主要是基于osi的传输层和包转发
     7层负载均衡主要是应用层 http、header、rewrite的改写
     nginx是属于7层的LSB
 
 14. upstream name {...}
     upstream name {
       server server1 backup; backup代表备用节点
       server server2 down; down代表不参与
       max_fails num; 允许请求失败的次数
       fails_timeout; 经过max_fails次失败后、服务暂停的时间
     }
     
 15. nginx轮询策略
     默认是基于请求(轮询、加权轮询)
     可以基于ip (ip_hash: 来自同一ip的请求会到达同一server、若前边还有一次负载均衡、得到的不是user真实的ip)
     可以基于url 
     last_conn (最少连接数、那个server的连接数少、达到哪个server上)
     hash 关键字
 
 16. 代理缓存
     proxy_cache_path path levels:levels 存放对应的缓存文件 eg: 1:2表示2级目录
     proxy_cache zone; 
     proxy_cache_valid 200 304 2m;说明对于状态为200和304的缓存文件的缓存时间是2分钟，两分钟之后再访问该缓存文件时，文件会过期，从而去源服务器重新取数据。
     proxy_Cache_valid [code];
     对缓存的过期与清除起作用的因素的优先级从高到低依次为：
inactive配置项、源服务器设置的Expires、源服务器设置的Max-Age、proxy_cache_valid配置项

17. proxy_next_upstream error timeout invalid_header http_500 http_502 http_503 http_504 ;
    表示若上游返回的状态码是error或者其它列出的状态、就自动连接下一台server

18. proxy_no_Cache string; 配置哪些不缓存

19. 大文件分片请求、 http_slice_module
    优势：每个子请求收到的数据都会形成独立的文件、一个请求中断另外的请求不受影响
    缺点：文件很大或者slice很小时可能会造成文件描述符耗尽
20. rewrite 
    1) url跳转、支持开发者设计
    2) seo优化
       rewrite ^(.*)$ /pages/index.html break; 将所有请求转发到pages/index.html页面
       语法：rewrite regex replacement [flag];
       flag有：
       last匹配到规则之后会跳转对应location、类似重新发起一次回话的效果
       break匹配到之后、直接终止下边的规则匹配
       redirect 返回302临时重定向,下次访问还会请求后端server
       permannent 301永久重定向、brower会记录、下次访问同样url直接跳转，不向后端发请求
     rewrite的优先级：
     server -> location -> location中选中的rewrite
    
```



