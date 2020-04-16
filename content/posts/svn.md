---
title: Subversion 安装与配置
date: 2018-11-24
categories:
  - linux
tags:
  - linux
  - svn
---

Subversion 安装与配置
<!--more-->

## 1. 安装 Subversion

```
yum install -y subversion
```

## 2. 创建 SVN 版本库

```
mkdir -p /data/svn/myproject
svnadmin create /data/svn/myproject
```

## 3. 配置 SVN 信息
### 配置文件简介
版本库中的配置目录 conf 有三个文件:

- authz 是权限控制文件  
- passwd 是帐号密码文件  
- svnserve.conf 是SVN服务综合配置文件  

配置权限配置文件 authz
**示例代码：/data/svn/myproject/conf/authz**

```conf
[groups]            
#用户组
admin = admin,root,test  
#用户组所对应的用户
[/]                 
#库目录权限
@admin = rw         
#用户组权限
*=r               
#非用户组权限
```

配置账号密码文件 passwd
**示例代码：/data/svn/myproject/conf/passwd**

```conf
[users]
# harry = harryssecret
# sally = sallyssecret
admin = 123456
root = 123456
test = 123456
```

配置 SVN 服务综合配置文件 svnserve.conf
**示例代码：/data/svn/myproject/conf/svnserve.conf**

```conf
[general]
# force-username-case = none
# 匿名访问的权限 可以是read、write，none，默认为read
anon-access = none
#使授权用户有写权限
auth-access = write
#密码数据库的路径
password-db = passwd
#访问控制文件
authz-db = authz
#认证命名空间，SVN会在认证提示里显示，并且作为凭证缓存的关键字
realm = /data/svn/myproject

[sasl]
```

## 4. 启动 SVN 服务
### 启动 SVN
```
svnserve -d -r /data/svn
```

### checkout SVN项目

```
mkdir -p /data/workspace/myproject
svn co svn://127.0.0.1/myproject /data/workspace/myproject --username root --password 123456 --force --no-auth-cache
```

### 提交文件到 SVN 服务器

从本地提交文件到 SVN 服务器，其中 root 密码为 /data/svn/myproject/conf/passwd 文件存储的密码

```
cd /data/workspace/myproject
echo test >> test.txt
svn add test.txt
svn commit test.txt -m 'test'
```

提交成功后可以通过如下命令从本地项目删除文件

```
cd /data/workspace/myproject
rm -rf test.txt
```

删除后可以通过 SVN 服务器恢复

```
cd /data/workspace/myproject
svn update
```


清除SVN客户端密码方法：
邮件选择TortoiseSVN中的settings选项---Saved Data---右边会发现有个Authentication data,点击clear，即完成清空账号密码操作，确认即可。下次就会自动弹出登录框。

在eclipse的svn插件中，我们也会保存密码，那么要清除时，换账号登录，就稍微麻烦点：

1. 查看你的Eclipse中使用的是什么SVN 接口
windows > preference > Team > SVN

2. 如果是用的JavaHL, 找到以下目录并删除auth目录.
C:\Documents and Settings\User\Application Data\Subversion\

3. 如果你用的SVNKit, 找到以下目录并删除.keyring文件.
eclipse\configuration\org.eclipse.core.runtime\