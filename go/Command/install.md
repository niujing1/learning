###go install

用于编译并按照指定的代码包和它们的依赖，会先处理依赖包
实际上，`go install`相当于`go build` + `安装编译后的结果文件到指定目录`

如果在执行`go install`时不添加代码包参数，命令将试图编译当前目录对应的代码包
编译代码包使用了与`go build`命令相同的程序

`go install`会把文件安装到对应的位置，只有包含main函数的文件才会生成可执行文件，其它的都会被当成代码包放到`pkg`目录

**要记得设置`GOBIN`**不然执行`go install`的时候会报错，找不到源码目录
`no buildable Go source files in /Users/nj/www/Go/src`

