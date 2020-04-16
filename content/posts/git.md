---
title: Git 指南
date: 2019-05-26
categories:
  - tools
tags:
  - git
---

Git 概念以及常用命令
<!--more-->

## 1. 获取 Git 仓库
### 在现有目录中初始化仓库
~~~bash
git init
~~~

### 克隆现有的仓库
~~~bash
git clone https://github.com/libgit2/libgit2 mylibgit
~~~


## 2. 记录每次更新到仓库

### 检查当前文件状态
~~~bash
git status
~~~
### 跟踪新文件
~~~bash
git add README
git status
~~~
### 暂存已修改文件
### 状态简览
~~~bash
git status --short
git status -s
~~~
### 忽略文件
使用 .gitignore 文件
-  所有空行或者以 ＃ 开头的行都会被 Git 忽略。
- 可以使用标准的 glob 模式匹配。
- 匹配模式可以以（/）开头防止递归。
- 匹配模式可以以（/）结尾指定目录。
- 要忽略指定模式以外的文件或目录，可以在模式前加上惊叹号（!）取反。

glob 模式
- 星号（*）匹配零个或多个任意字符；
- [abc] 匹配任何一个列在方括号中的字符；
- 问号（?）只匹配一个任意字符；
- 如果在方括号中使用短划线分隔两个字符，表示所有在这两个字符范围内的都可以匹配；
- 使用两个星号（*) 表示匹配任意中间目录。

### 查看已暂存和未暂存的修改
- 查看尚未暂存的文件更新了哪些部分

~~~bash
git diff
~~~


- 查看已暂存的将要添加到下次提交里的内容

~~~bash
git diff --staged
git diff --cached
~~~

### 提交更新
~~~bash
git commit -m "first commit"
~~~

### 跳过使用暂存区域
~~~bash
git commit -a -m "second commit"
~~~

### 移除文件
~~~bash
git rm [file_name]
~~~

把文件从 Git 仓库中删除，文件保留在当前工作目录中
~~~bash
git rm --cached [file_name]
~~~

### 移动文件
~~~bash
git mv file_from file_to
~~~



## 3. 查看提交历史

~~~bash
git log
~~~


~~~bash
git log -p -2 # -p 显示每次提交的内容差异，-2 仅显示最近两次提交
git log --stat # 查看提交的简略的统计信息
git log --oneline # 将每个提交放在一行显示
git log --short
git log --full
git log --fuller
git log --pretty=online
git log --pretty=format:"%h - %an, %ar : %s" --graph # 定制要显示的记录格式
~~~

`git log` 常用选项

|            选项 | 说明                                                         |
| --------------: | :----------------------------------------------------------- |
|              -p | 按补丁格式显示每个更新之间的差异                             |
|          --stat | 显示每次更新的文件修改统计信息。                             |
|     --shortstat | 只显示 --stat 中最后的行数修改添加移除统计。                 |
|     --name-only | 仅在提交信息后显示已修改的文件清单。                         |
|   --name-status | 显示新增、修改、删除的文件清单。                             |
| --abbrev-commit | 仅显示 SHA-1 的前几个字符，而非所有的 40 个字符。            |
| --relative-date | 使用较短的相对时间显示（比如，“2 weeks ago”）。              |
|         --graph | 显示 ASCII 图形表示的分支合并历史。                          |
|        --pretty | 使用其他格式显示历史提交信息。可用的选项包括 oneline，short，full，fuller 和 format（后跟指定格式）。 |

`git log --pretty=format` 常用的选项

|选项|说明|
|--:|:--|
|%H|提交对象（commit）的完整哈希字串|
|%h|提交对象的简短哈希字串|
|%T|树对象（tree）的完整哈希字串|
|%t|树对象的简短哈希字串|
|%P|父对象（parent）的完整哈希字串|
|%p|父对象的简短哈希字串|
|%an|作者（author）的名字|
|%ae|作者的电子邮件地址|
|%ad|作者修订日期（可以用 --date= 选项定制格式）|
|%ar|作者修订日期，按多久以前的方式显示|
|%cn|提交者（committer）的名字|
|%ce|提交者的电子邮件地址|
|%cd|提交日期|
|%cr|提交日期，按多久以前的方式显示|
|%s|提交说明|

### 限制输出长度

~~~bash
git log --since=2.weeks
~~~

限制 `git log` 输出的选项

|              选项 | 说明                               |
| ----------------: | :--------------------------------- |
|              -(n) | 仅显示最近的 n 条提交              |
|  --since, --after | 仅显示指定时间之后的提交           |
| --until, --before | 仅显示指定时间之前的提交           |
|          --author | 仅显示指定作者相关的提交           |
|       --committer | 仅显示指定提交者相关的提交         |
|            --grep | 仅显示含指定关键字的提交           |
|                -S | 仅显示添加或移除了某个关键字的提交 |



## 4. 撤消操作

~~~bash
git commit --amend
~~~

### 取消暂存的文件

~~~bash
# git reset HEAD <file> ...
git reset HEAD CONTRIBUTING.md
~~~

### 撤消对文件的修改

~~~bash
# git checkout -- <file>
git checkout -- CONTRIBUTING.md
~~~



## 5. 远程仓库的使用

~~~bash
git clone https://github.com/schacon/ticgit
git remote
git remote -v
git remote add pb https://github.com/paulboone/ticgit
git fetch pb
~~~

### 从远程仓库中抓取与拉取

~~~bash
git fetch [remote-name]
~~~

### 推送到远程仓库

~~~bash
git push origin master
~~~

### 查看远程仓库

~~~bash
# git remote show [remote-name]
git remote show origin
~~~

### 远程仓库的移除与重命名

~~~bash
git remote rename pb paul
git remote rm paul
~~~



## 6. 打标签

### 列出标签

~~~bash
git tag
git tag -l 'v1.8.5*'
~~~

### 创建标签

### 附注标签

~~~bash
git tag -a v1.4 -m 'my version 1.4'
git tag
git show v1.4 # 查看标签信息与对应的提交信息
~~~

### 轻量标签

~~~bash
git tag v1.4-lw
git tag
~~~

### 后期打标签

~~~bash
git log --pretty=oneline
git tag -a v1.2 <部分校验和> # git tag -a v1.2 9fceb02
~~~

### 共享标签

~~~bash
git push origin v1.5
git push origin --tags # 一次性推送全部标签
~~~

### 检出标签

~~~bash
# 使用 git checkout -b [branchname] [tagname] 在特定的标签上创建一个新分支
git checkout -b version2 v2.0.0
~~~



## 7. Git 别名

~~~bash
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.st status
git config --global alias.unstage 'reset HEAD --'
git config --global alias.last 'log -1 HEAD'
# 执行外部命令，在命令前面加入 ! 符号
git config --global alias.visual '!gitk'
~~~

