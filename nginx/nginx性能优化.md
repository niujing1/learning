#### nginx性能优化：

```
nginx性能优化：

1. 性能优化考虑点：
   1）当前系统结构瓶颈  观察指标、压力测试
   2）了解业务模式  接口业务类型、系统层次化结构
   3) 性能与安全  
   
2. 压测工具ab
   ab -n 200 -c 20 http://www.baidu.com
   1) -n 请求总数
   2) -c 并发请求的数量
   3) -k 是否开启长连接
   
   结果解释
   Concurrency Level 并发的级别
   Time taken for tests 总消耗的时间
   Complete requests 完成的请求总数
   Failed requests 失败的请求数
   Requests per second 每秒的请求数
   Time per request 单个请求所需要的时间(站在客户端的角度)
   Time per request 单个请求所需要的时间(站在server的角度、只包含server处理的时间)
   Transfer rate 传输速率

3. 系统与nginx的性能优化
   1）网络
   2）系统
   3）服务
   4）程序本身
   5）数据库、底层服务
   要知道哪一层是系统性能的短板
   文件句柄
```

