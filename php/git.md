### 工作中遇到的一些git需求

当前仓库的修改在.git/config中  全局修改在~/.gitconfig文件中

```
git config --global alias.st status  缩写
git log -p commitid  查看某次commit的修改
git log -p --author=name  查看指定用户的修改
git commit  --amend 修改某次提交的备注
