###使用vim搭建go环境


```
1. 安装gotags
go get -u github.com/jstemmer/gotags

2. 使用bundle安装tagbar
在 ~/.vimrc文件中添加 Plugin 'Tagbar'

然后 执行 sudo vim +PluginInstall 安装插件，直到出现Done表示安装完成

3. 设置tagbar
" 设置tagbar的窗口宽度 
let g:tagbar_width=30
" 映射Tagbar的快捷键,按Ctrl+m自动打开 
map <C-m> :TagbarToggle<CR>

4. 把gotags所在的环境变量添加到PATH中
5. 进入项目目录，执行 gotags -R project/*.go > tags
6. 测试使用：
打开一个文件，按下 ctrl+m 可以看到侧边栏打开，列出所有函数、导入文件、结构变量等
再次按下 ctrl+m 可以隐藏
  
函数跳转的使用同cscope
ctrl+] : 跳转到函数定义处
ctrl+t : 跳回原处

```

