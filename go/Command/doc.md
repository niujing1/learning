### go doc && godoc

`go doc`打印go程序实体文档(包括常量、变量、函数、结构体及接口)

选项 | 作用
-----|-----------
-c   | 使go doc 区分大小写，默认不区分
-cmd | 同时打印main包终端额可导出实体
-u   | 同时打印不可导出的程序实体默认不导出

**注意** `go doc`若当前代码包与标准库中的包重名，会只打印第一个匹配的包

`godoc`

```
godoc -src fmt Printf 同时打印出源代码
godoc -ex 可以同时打印出示例用法
godoc -http=:6060 表示启动的web服务器使用本机的6060端口，之后可以使用localhost:6060来用网页查看go的文档
godoc -http=:9090 -index 表示开启搜索索引
godoc -q -server="192.168.1.4:9090" Listener 如果在godoc中开启了Go的web服务器，且ip为192.168.1.4 port为9090就可以在终端进行查询
```
