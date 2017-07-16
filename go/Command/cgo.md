### go tool cgo

cgo也是一个Go语言自带的特殊工具。一般情况下，我们使用命令go tool cgo来运行它。这个工具可以使我们创建能够调用C语言代码的Go语言源码文件。这使得我们可以使用Go语言代码去封装一些C语言的代码库，并提供给Go语言代码或项目使用。

在执行`go tool cgo`命令的时候，我们需要加入作为目标的Go语言源码文件的路径。这个路径可以是绝对路径也可以是相对路径。但是，作者强烈建议在目标源码文件所属的代码包目录下执行`go tool cgo`命令并以目标源码文件的名字作为参数。因为，`go tool cgo`命令会在当前目录（也就是我们执行`go tool cgo`命令的目录）中生成一个名为`_obj`的子目录。该目录下会包含一些Go源码文件和C源码文件。这个子目录及其包含的文件理应被保存在目标代码包目录之下。

