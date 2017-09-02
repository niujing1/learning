### git 使用

一、源起：

```
	可能基本使用大家都比较清楚、但是关于一些细节、可能很少人知道其实现，一些错误发生时不知道如何回退,简单梳理了一下、希望可以有所帮助~~
  
```

二、背景

```
文本和代码项目的备份策略，通常包括版本控制、或者称为"变更追踪管理",这些持续的变更可以称为版本库，代码管理工具通常称为VCS(版本控制系统)、SCM(源码控制管理)、或者RCS(修订控制系统)、最初只是Linus Torvalds为了方便管理Linux内核的代码开发工作、如今由于它的功能强大并且开销比较低已在大量项目中广泛应用
```

三、基本概念：

```
git主要维护2种数据结构：对象库(object store)和索引(index)
git对象包括：块blob、目录树tree、提交commit、标签tag

一个blob包含一个文件的数据、但不包含元数据、甚至文件名也没有
一个tree保护一层目录信息
一个commit保护一次变化的元数据、包括作者、提交日志和时间
一个tag就是一个特定对象、通常是commit

索引是一个临时的、动态二进制文件 .git/index

git存储的是文件数据的sha1值
每次版本变更存储的是完整数据、而不是diff

git write-tree 捕获当前的索引状态
git ls-files -s 列出索引中包含的文件
git rev-parse 查找完整commit | tag eg. git rev-parse f761ec
git cat-file -p commit | tag   eg. git cat-file -p f761ec

```

git文件：

```
git 中的文件分为3种：tracked、untracked、ignored

git add 把文件放进索引
git commit 提交暂存区文件
git commit --all 递归查找暂存所有已知的和已修改的文件、并提交
git rm file 删除一个文件
git rm --cached 移出暂存区、保留工作目录的文件
git checkout HEAD -- c.txt 将删除的文件恢复
git log file 查看一个文件的提交历史
git log --follow file 追踪查看一个文件的提交历史
```

.gitignore文件
```
1. 空行会被忽略
2. #开头表示注释行
3. /结尾表示目录
4. shell通配符可扩展 eg 、test/*.php 会查找test目录下所有php文件
5. 开头的! 代表模式取反
```

引用refs
```
每个引用都有一个ref开头的明确全称、放在.git/refs/下
eg、dev的本地分支其实就是 .git/refs/heads/dev的缩写

HEAD 指向最新提交
ORIG_HEAD merge会把先前的head记录到ORGI_HEAD中、可以使用它回复原状态
FETCH_HEAD 最近抓取的远程分支HEAD
MERGE_HEAD merge操作进行时，正在进行合并进head的提交
```

分支 & 提交
```
git show-branch 查看分支提交历史
git log -1 --pretty=oneline HEAD 查看当前头指针指向的commit
git log branchname 查看某一个分支的提交
git log commit 查看某一次commit的提交
git log -p commit | branchname 查看某一个提交或者分支的文件改动
git log -p filename 查看某个文件的改动历史
git show commit^2 同一代提交的不同父提交
git show commit~2 上一代提交
git blame filename 查看某一个文件的详细改动历史


git branch 查看分支
git show-branch

