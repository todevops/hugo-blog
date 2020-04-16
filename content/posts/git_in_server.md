---
title: 在服务器上搭建 Git
date: 2019-12-20
lastmod: 2019-12-20
categories:
  - tools
tags:
  - git
---

[服务器上的 Git](https://git-scm.com/book/zh/v2/%E6%9C%8D%E5%8A%A1%E5%99%A8%E4%B8%8A%E7%9A%84-Git-%E5%9C%A8%E6%9C%8D%E5%8A%A1%E5%99%A8%E4%B8%8A%E6%90%AD%E5%BB%BA-Git) - 在服务器上搭建 Git
<!--more-->

## 在服务器上创建裸仓库

假设一个域名为 `git.example.com` 的服务器已经架设好，并可以通过 SSH 连接，你想把所有的 Git 仓库放在 /opt/git 目录下。 

```bash
# 创建 git 用户
[root@git ~]# useradd git -m -d /opt/git

[root@git ~]# echo "git" | passwd --stdin git
更改用户 git 的密码 。
passwd：所有的身份验证令牌已经成功更新。

[root@git ~]# su - git
[git@git ~]$ pwd
/opt/git


[git@git ~]$ git init --bare project.git
初始化空的 Git 版本库于 /opt/git/project.git/
```

## 客户端配置连接

### 客户端

如果一个用户，通过使用 SSH 连接到一个服务器，并且其对 `/opt/git/project.git` 目录拥有可写权限，那么他将自动拥有推送权限。（这里我们默认使用 git 用户）

```bash
ssh-copy-id git@git.example.com

git clone git@git.example.com:/opt/git/project.git

正克隆到 'project'...
warning: 您似乎克隆了一个空仓库。

cd project

echo "## project" > README.md
git add .
git commit -m "initial new project"

[master（根提交） 1304197] initial new project
 1 file changed, 1 insertion(+)
 create mode 100644 README.md

git push

枚举对象: 3, 完成.
对象计数中: 100% (3/3), 完成.
写入对象中: 100% (3/3), 226 字节 | 226.00 KiB/s, 完成.
总共 3 （差异 0），复用 0 （差异 0）
To git.example.com:/opt/git/project.git
 * [new branch]      master -> master
```
### 服务器端

返回服务端查看文件变化

```bash
[git@git ~]$ ls -1 project.git/
branches
config
description
HEAD
hooks
info
objects
refs
```