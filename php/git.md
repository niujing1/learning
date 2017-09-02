### 工作中遇到的一些git需求


1. 配置文件

```
git 的配置文件分为3级：系统system 用户global 目录local
系统级：/etc/gitconfig git config --system user.name "nj"
        或者/usr/local/etc/gitconfig
用户级：~/.gitconfig  git config --global user.name "nj"
目录级：.git/config  git config --local user.name "nj"

当前仓库的修改在.git/config中  全局修改在~/.gitconfig文件中

设置用户名和邮箱
git config user.name niujing
git config user.email niujing@lianjia.com

git config -l 列出所有的设置项
git config --unset user.nam 删除一个配置项
```

2. 基本使用：

```
添加文件到暂存区（staged）：       git add filename / git stage filename
将当前文件夹及子目录的文件都添加到版本库：git add .
撤回暂存区文件：git rm --cached file
将所有修改文件添加到暂存区：git add --all / git add -A
提交修改到暂存区（staged）：
git commit -m 'commit message' / git commit -a -m 'commit message'
从git仓库删除文件：git rm filename
从Git仓库中删除文件，但本地文件保留：git rm --cached filename

git config --global alias.st status  缩写
git log -p commitid  查看某次commit的修改
git log -p --author=name  查看指定用户的修改
git commit  --amend 修改某次提交的备注
git rm --cached filename 从Git仓库中删除文件，但本地文件保留
git rm --cached filename 删除git和本地
git mv filename newfilename 重命名文件
git pull (origin branchname) 可以指定分支拉取，也可以忽略
pull会自动fetch代码并merge，提示冲突
```

```
检出（clone）仓库代码：git clone repository-url
查看远程仓库：git remote -v
添加远程仓库：git remote add [name] [repository-url]
删除远程仓库：git remote rm [name]
修改远程仓库地址：git remote set-url origin new-repository-url
拉取远程仓库： git pull [remoteName] [localBranchName]
推送远程仓库： git push [remoteName] [localBranchName] 例: git push -u orgin master 将当前分支推送到远端master分支
将本地 test 分支提交到远程 master 分支: git push origin test:master 
(把本地的某个分支 test 提交到远程仓库，并作为远程仓库的 master 分支) 
提交本地 test 分支作为远程的 test 分支 :git push origin test:test

```



diff :

```
git diff commit1 commit2 查看2次提交的差异

```


add :
```
```

rm :
```
```

tag :
```
git tag -m "tag"
```


commit :
```

```
