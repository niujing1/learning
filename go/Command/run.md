### go run

`go run`编译并运行go命令源码文件，不接受代码包
`go run test.go -p paramName` -p表示是一个参数名而不是参数值
`go run -n hello.go`只打印运行过程需要执行的命令，并不实际执行
`go run` 会把-p后边的参数原封不动的传递给对应的可执行文件

