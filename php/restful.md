#### restful API

http 协议响应说明

```
200 OK
400 Bad Request 客户端请求语法有误、不能被服务端正确解析
401 Unauthorized 服务端收到请求、但是需要认证、所以拒绝提供服务
404 Not Fount 未找到资源
500 Internal Server Error 服务器发生错误
```

soap 由于各种需求不断扩充本身的协议内容、导致性能下降、学习成本变高

```
rpc: 远程调用、面向方法
soap：面向服务、面向消息
rest：面向资源

