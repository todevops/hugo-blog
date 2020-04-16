---
title: uWSGI 部署 Django 应用
date: 2019-01-07
categories:
  - linux
tags:
  - linux
  - python
---

uWSGI 部署 Django 应用
<!--more-->
## 准备项目代码
```bash
mkdir -p /appdata/uwsgi
cd /appdata/uwsgi
# 将项目代码放置此处
```

## 安装相关工具和依赖

```bash
[root@uwsgi ~]# yum -y install git wget httpd vim # 安装相关工具
[root@uwsgi ~]# yum -y install gcc zlib* openssl-devel # 安装编译工具和依赖库
```
## 编译安装 Python 环境
rhel系列无法通过yum直接安装python3，需要源码编译安装
```bash
[root@uwsgi ~]# pwd # 查看当前目录
/root
[root@uwsgi ~]# wget https://www.python.org/ftp/python/3.6.5/Python-3.6.5.tgz # 下载python3.6.5
[root@uwsgi ~]# tar -zxvf Python-3.6.5.tgz # 解压
[root@uwsgi ~]# cd Python-3.6.5

[root@uwsgi Python-3.6.5]# pwd # 查看当前目录
/root/Python-3.6.5
[root@uwsgi Python-3.6.5]# ./configure # 编译
[root@uwsgi Python-3.6.5]# make && make install # 安装
[root@uwsgi Python-3.6.5]# which python3 # 查看python3的路径
/usr/local/bin/python3
```

# 使用 epel 源 Python 环境
```bash
[root@uwsgi ~]# yum -y install epel-release
[root@uwsgi ~]# yum makecache
[root@uwsgi ~]# yum -y install python3 python3-devel
````
+ 出现以下报错为缺少 python3-devel
> *** uWSGI compiling embedded plugins ***  
    [gcc -pthread] plugins/python/python_plugin.o  
    In file included from plugins/python/python_plugin.c:1:0:  
    plugins/python/uwsgi_python.h:2:20: 致命错误：Python.h：没有那个文件或目录  
    #include <Python.h>

## 安装 uWSGI
```bash
[root@uwsgi ~]# python3 -m pip install uwsgi
```
## 安装配置虚拟环境
```bash
[root@uwsgi ~]# python3 -m pip install virtualenv # 安装虚拟环境
[root@uwsgi ~]# python3 -m virtualenv /appdata/uwsgi/venv # 配置虚拟环境
[root@uwsgi ~]# source /venv/bin/activate # 激活虚拟环境
(venv) [root@uwsgi ~]# pip install -r /appdata/planner/requirements.txt # 安装项目依赖
(venv) [root@uwsgi ~]# pip freeze # 查看依赖库是否安装
(venv) [root@uwsgi ~]# deactivate # 取消激活虚拟环境
[root@uwsgi ~]# 
```

## 修改 ALLOW_HOSTS
```bash
[root@uwsgi ~]# vim /appdata/planner/planner/settings.py # 编辑项目中的 settings.py
```
```py
ALLOWED_HOSTS = ["*"]
```

## 配置 uWSGI
```bash
[root@uwsgi ~]# mkdir -p /applog/uwsgi/ # 创建 uwsgi 的日志目录
[root@uwsgi ~]# vim /etc/uwsgi/planner.ini # 创建 uwsgi.ini 配置文件
```

```ini
[uwsgi]
# 项目目录
chdir=/appdata/uwsgi/planner
http=192.168.33.11:8000
# 虚拟环境目录
home=/appdata/uwsgi/venv
module=planner.wsgi:application
master=True
pidfile=/applog/uwsgi/planner.pid
daemonize=/applog/uwsgi/planner.log
socket=/applog/uwsgi/planner.sock
max-requests=5000
env=LANG=en_US.UTF-8
buffer-size=32768
logformat={"uri": "%(uri)",  "method": "%(method)",  "user": "%(user)",  "addr": "%(addr)",  "host": "%(host)",  "proto": "%(proto)",  "uagent": "%(uagent)",  "referer": "%(referer)",  "status": "%(status)",  "micros": "%(micros)",  "msecs": "%(msecs)",  "time": "%(time)",  "ctime": "%(ctime)",  "epoch": "%(epoch)",  "size": "%(size)",  "ltime": "%(ltime)",  "hsize": "%(hsize)",  "rsize": "%(rsize)",  "cl": "%(cl)",  "pid": "%(pid)",  "wid": "%(wid)",  "switches": "%(switches)",  "vars": "%(vars)",  "headers": "%(headers)",  "core": "%(core)",  "vsz": "%(vsz)",  "rss": "%(rss)",  "vszM": "%(vszM)",  "rssM": "%(rssM)",  "pktsize": "%(pktsize)",  "modifier1": "%(modifier1)",  "modifier2": "%(modifier2)",  "metric": "%(metric.XXX)",  "rerr": "%(rerr)",  "werr": "%(werr)",  "ioerr": "%(ioerr)",  "tmsecs": "%(tmsecs)",  "tmicros": "%(tmicros)"}
```

## 启动 uWSGI
```bash
# 以 /etc/uwsgi.ini 配置启动 uwsgi
[root@uwsgi ~]# uwsgi --init /etc/uwsgi.ini

# 查看端口是否监听
[root@uwsgi ~]# netstat -ntlp | grep uwsgi
tcp        0      0 192.168.33.11:8000      0.0.0.0:*               LISTEN      13162/uwsgi
```
## 配置 Nginx
```bash
# 安装 nginx
[root@uwsgi ~]# yum -y install epel-release
[root@uwsgi ~]# yum -y install nginx
# 创建配置文件
[root@uwsgi ~]# cat << EOF > /etc/nginx/conf.d/planner.conf
server {
	listen 80;
	server_name 192.168.33.11;
	charset utf-8;
	location /static {
		alias /appdata/uwsgi/planner/static/;
	}
	location /media  {
		alias /appdata/uwsgi/planner/media/;
	}
	location /{
		include /etc/nginx/uwsgi_params;
		uwsgi_pass unix:/applog/uwsgi/planner.sock;
	}
}
EOF
# 检查文件是否合法
[root@uwsgi ~]# nginx -t
# 刷新配置文件
[root@uwsgi ~]# nginx -s reload
```
现在就可以使用 http://<IP> 访问 Django 应用了

## 注意事项

未安装zlib*：
```
zipimport.ZipImportError: can't decompress data; zlib not available # 编译时报错
```
未安装openssl、openssl-devel：
```
pip is configured with locations that require TLS/SSL, however the ssl module in Python is not available # pip 安装 uwsgi 时报错
```
uwsgi 无法访问，日志报错：
```
invalid request block size: 21573 (max 4096)...skip # 在 /etc/uwsgi.ini 中添加 buffer-size=32768
```