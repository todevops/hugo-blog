---
title: Nginx 安装与配置
date: 2019-12-31
categories:
  - linux
tags:
  - nginx
---

Nginx 的安装以及相关配置
<!--more-->


## Nginx 安装
### 1. yum 源安装
+ epel 源

```bash
yum -y install epel-release
yum makecache
yum -y install nginx

systemctl enable nginx && systemctl start nginx
```
+ nginx 官方源
[http://nginx.org/en/linux_packages.html#RHEL-CentOS](http://nginx.org/en/linux_packages.html#RHEL-CentOS)

```bash
yum install yum-utils


cat < EOF > /etc/yum.repos.d/nginx.repo
[nginx-stable]
name=nginx stable repo
baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
gpgcheck=1
enabled=0
gpgkey=https://nginx.org/keys/nginx_signing.key
module_hotfixes=true

[nginx-mainline]
name=nginx mainline repo
baseurl=http://nginx.org/packages/mainline/centos/$releasever/$basearch/
gpgcheck=1
enabled=0
gpgkey=https://nginx.org/keys/nginx_signing.key
module_hotfixes=true
EOF

yum-config-manager --enable nginx-stable
yum-config-manager --enable nginx-mainline

yum makecache
yum install nginx
systemctl enable nginx && systemctl start nginx
```

### 2. 源码编译安装

```bash
[root@nginx nginx-1.16.1]# yum -y install gcc pcre-devel zlib-devel
```

下载 nginx 源码[http://nginx.org/en/download.html](http://nginx.org/en/download.html)

```bash
[root@nginx ~]# wget http://nginx.org/download/nginx-1.16.1.tar.gz
[root@nginx ~]# tar -zxvf nginx-1.16.1.tar.gz
[root@nginx ~]# cd nginx-1.16.1
```

- configure 参数

> 根据需求设置编译参数

```bash
[root@nginx nginx-1.16.1]# ./configure --help

  --help                             print this message

  --prefix=PATH                      set installation prefix
  --sbin-path=PATH                   set nginx binary pathname
  --modules-path=PATH                set modules path
  --conf-path=PATH                   set nginx.conf pathname
  --error-log-path=PATH              set error log pathname
  --pid-path=PATH                    set nginx.pid pathname
  --lock-path=PATH                   set nginx.lock pathname

  --user=USER                        set non-privileged user for
                                     worker processes
  --group=GROUP                      set non-privileged group for
                                     worker processes

  --build=NAME                       set build name
  --builddir=DIR                     set build directory

  --with-select_module               enable select module
  --without-select_module            disable select module
  --with-poll_module                 enable poll module
  --without-poll_module              disable poll module

  --with-threads                     enable thread pool support

  --with-file-aio                    enable file AIO support

  --with-http_ssl_module             enable ngx_http_ssl_module
  --with-http_v2_module              enable ngx_http_v2_module
  --with-http_realip_module          enable ngx_http_realip_module
  --with-http_addition_module        enable ngx_http_addition_module
  --with-http_xslt_module            enable ngx_http_xslt_module
  --with-http_xslt_module=dynamic    enable dynamic ngx_http_xslt_module
  --with-http_image_filter_module    enable ngx_http_image_filter_module
  --with-http_image_filter_module=dynamic
                                     enable dynamic ngx_http_image_filter_module
  --with-http_geoip_module           enable ngx_http_geoip_module
  --with-http_geoip_module=dynamic   enable dynamic ngx_http_geoip_module
  --with-http_sub_module             enable ngx_http_sub_module
  --with-http_dav_module             enable ngx_http_dav_module
  --with-http_flv_module             enable ngx_http_flv_module
  --with-http_mp4_module             enable ngx_http_mp4_module
  --with-http_gunzip_module          enable ngx_http_gunzip_module
  --with-http_gzip_static_module     enable ngx_http_gzip_static_module
  --with-http_auth_request_module    enable ngx_http_auth_request_module
  --with-http_random_index_module    enable ngx_http_random_index_module
  --with-http_secure_link_module     enable ngx_http_secure_link_module
  --with-http_degradation_module     enable ngx_http_degradation_module
  --with-http_slice_module           enable ngx_http_slice_module
  --with-http_stub_status_module     enable ngx_http_stub_status_module

  --without-http_charset_module      disable ngx_http_charset_module
  --without-http_gzip_module         disable ngx_http_gzip_module
  --without-http_ssi_module          disable ngx_http_ssi_module
  --without-http_userid_module       disable ngx_http_userid_module
  --without-http_access_module       disable ngx_http_access_module
  --without-http_auth_basic_module   disable ngx_http_auth_basic_module
  --without-http_mirror_module       disable ngx_http_mirror_module
  --without-http_autoindex_module    disable ngx_http_autoindex_module
  --without-http_geo_module          disable ngx_http_geo_module
  --without-http_map_module          disable ngx_http_map_module
  --without-http_split_clients_module disable ngx_http_split_clients_module
  --without-http_referer_module      disable ngx_http_referer_module
  --without-http_rewrite_module      disable ngx_http_rewrite_module
  --without-http_proxy_module        disable ngx_http_proxy_module
  --without-http_fastcgi_module      disable ngx_http_fastcgi_module
  --without-http_uwsgi_module        disable ngx_http_uwsgi_module
  --without-http_scgi_module         disable ngx_http_scgi_module
  --without-http_grpc_module         disable ngx_http_grpc_module
  --without-http_memcached_module    disable ngx_http_memcached_module
  --without-http_limit_conn_module   disable ngx_http_limit_conn_module
  --without-http_limit_req_module    disable ngx_http_limit_req_module
  --without-http_empty_gif_module    disable ngx_http_empty_gif_module
  --without-http_browser_module      disable ngx_http_browser_module
  --without-http_upstream_hash_module
                                     disable ngx_http_upstream_hash_module
  --without-http_upstream_ip_hash_module
                                     disable ngx_http_upstream_ip_hash_module
  --without-http_upstream_least_conn_module
                                     disable ngx_http_upstream_least_conn_module
  --without-http_upstream_random_module
                                     disable ngx_http_upstream_random_module
  --without-http_upstream_keepalive_module
                                     disable ngx_http_upstream_keepalive_module
  --without-http_upstream_zone_module
                                     disable ngx_http_upstream_zone_module

  --with-http_perl_module            enable ngx_http_perl_module
  --with-http_perl_module=dynamic    enable dynamic ngx_http_perl_module
  --with-perl_modules_path=PATH      set Perl modules path
  --with-perl=PATH                   set perl binary pathname

  --http-log-path=PATH               set http access log pathname
  --http-client-body-temp-path=PATH  set path to store
                                     http client request body temporary files
  --http-proxy-temp-path=PATH        set path to store
                                     http proxy temporary files
  --http-fastcgi-temp-path=PATH      set path to store
                                     http fastcgi temporary files
  --http-uwsgi-temp-path=PATH        set path to store
                                     http uwsgi temporary files
  --http-scgi-temp-path=PATH         set path to store
                                     http scgi temporary files

  --without-http                     disable HTTP server
  --without-http-cache               disable HTTP cache

  --with-mail                        enable POP3/IMAP4/SMTP proxy module
  --with-mail=dynamic                enable dynamic POP3/IMAP4/SMTP proxy module
  --with-mail_ssl_module             enable ngx_mail_ssl_module
  --without-mail_pop3_module         disable ngx_mail_pop3_module
  --without-mail_imap_module         disable ngx_mail_imap_module
  --without-mail_smtp_module         disable ngx_mail_smtp_module

  --with-stream                      enable TCP/UDP proxy module
  --with-stream=dynamic              enable dynamic TCP/UDP proxy module
  --with-stream_ssl_module           enable ngx_stream_ssl_module
  --with-stream_realip_module        enable ngx_stream_realip_module
  --with-stream_geoip_module         enable ngx_stream_geoip_module
  --with-stream_geoip_module=dynamic enable dynamic ngx_stream_geoip_module
  --with-stream_ssl_preread_module   enable ngx_stream_ssl_preread_module
  --without-stream_limit_conn_module disable ngx_stream_limit_conn_module
  --without-stream_access_module     disable ngx_stream_access_module
  --without-stream_geo_module        disable ngx_stream_geo_module
  --without-stream_map_module        disable ngx_stream_map_module
  --without-stream_split_clients_module
                                     disable ngx_stream_split_clients_module
  --without-stream_return_module     disable ngx_stream_return_module
  --without-stream_upstream_hash_module
                                     disable ngx_stream_upstream_hash_module
  --without-stream_upstream_least_conn_module
                                     disable ngx_stream_upstream_least_conn_module
  --without-stream_upstream_random_module
                                     disable ngx_stream_upstream_random_module
  --without-stream_upstream_zone_module
                                     disable ngx_stream_upstream_zone_module

  --with-google_perftools_module     enable ngx_google_perftools_module
  --with-cpp_test_module             enable ngx_cpp_test_module

  --add-module=PATH                  enable external module
  --add-dynamic-module=PATH          enable dynamic external module

  --with-compat                      dynamic modules compatibility

  --with-cc=PATH                     set C compiler pathname
  --with-cpp=PATH                    set C preprocessor pathname
  --with-cc-opt=OPTIONS              set additional C compiler options
  --with-ld-opt=OPTIONS              set additional linker options
  --with-cpu-opt=CPU                 build for the specified CPU, valid values:
                                     pentium, pentiumpro, pentium3, pentium4,
                                     athlon, opteron, sparc32, sparc64, ppc64

  --without-pcre                     disable PCRE library usage
  --with-pcre                        force PCRE library usage
  --with-pcre=DIR                    set path to PCRE library sources
  --with-pcre-opt=OPTIONS            set additional build options for PCRE
  --with-pcre-jit                    build PCRE with JIT compilation support

  --with-zlib=DIR                    set path to zlib library sources
  --with-zlib-opt=OPTIONS            set additional build options for zlib
  --with-zlib-asm=CPU                use zlib assembler sources optimized
                                     for the specified CPU, valid values:
                                     pentium, pentiumpro

  --with-libatomic                   force libatomic_ops library usage
  --with-libatomic=DIR               set path to libatomic_ops library sources

  --with-openssl=DIR                 set path to OpenSSL library sources
  --with-openssl-opt=OPTIONS         set additional build options for OpenSSL

  --with-debug                       enable debug logging
```

```bash
[root@nginx nginx-1.16.1]# mkdir nginx-src
[root@nginx nginx-1.16.1]# ./configure --prefix=$PWD/nginx-src
[root@nginx nginx-1.16.1]# make && make install
[root@nginx nginx-1.16.1]# ./nginx-src/sbin/nginx -v
nginx version: nginx/1.16.1

[root@nginx nginx-1.16.1]# ldd ./nginx-src/sbin/nginx
	linux-vdso.so.1 =>  (0x00007fff44dea000)
	libdl.so.2 => /lib64/libdl.so.2 (0x00007f9d0b173000)
	libpthread.so.0 => /lib64/libpthread.so.0 (0x00007f9d0af57000)
	libcrypt.so.1 => /lib64/libcrypt.so.1 (0x00007f9d0ad20000)
	libpcre.so.1 => /lib64/libpcre.so.1 (0x00007f9d0aabe000)
	libz.so.1 => /lib64/libz.so.1 (0x00007f9d0a8a8000)
	libc.so.6 => /lib64/libc.so.6 (0x00007f9d0a4da000)
	/lib64/ld-linux-x86-64.so.2 (0x00007f9d0b377000)
	libfreebl3.so => /lib64/libfreebl3.so (0x00007f9d0a2d7000)

[root@nginx nginx-1.16.1]# mv nginx-src/ /opt/nginx
[root@nginx nginx-1.16.1]# ll /opt/nginx/
总用量 0
drwxr-xr-x 2 root root 333 1月   1 06:50 conf
drwxr-xr-x 2 root root  40 1月   1 06:50 html
drwxr-xr-x 2 root root   6 1月   1 06:50 logs
drwxr-xr-x 2 root root  19 1月   1 06:50 sbin
```

- 创建 nginx.service 文件

```bash
[root@nginx ~]# cat < EOF > nginx.service
[Unit]
Description=Nginx Server
After=network.target
After=network-online.target
Wants=network-online.target

[Service]
Type=forking
ExecStartPre=/opt/nginx/sbin/nginx -c /opt/nginx/conf/nginx.conf -p /opt/nginx -t
ExecStart=/opt/nginx/sbin/nginx -c /opt/nginx/conf/nginx.conf -p /opt/nginx
ExecReload=/opt/nginx/sbin/nginx -c /opt/nginx/conf/nginx.conf -p /opt/nginx -s reload
PrivateTmp=true
Restart=always
RestartSec=5
StartLimitInterval=0
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
EOF

[root@nginx ~]# cp nginx.service /etc/systemd/system/
[root@nginx ~]# systemctl enable nginx && systemctl start nginx

[root@nginx ~]# systemctl status nginx
● nginx.service - Nginx Server
   Loaded: loaded (/etc/systemd/system/nginx.service; disabled; vendor preset: disabled)
   Active: active (running) since 三 2020-01-01 06:59:14 CST; 5s ago
  Process: 10052 ExecStart=/opt/nginx/sbin/nginx -c /opt/nginx/conf/nginx.conf -p /opt/nginx (code=exited, status=0/SUCCESS)
  Process: 10051 ExecStartPre=/opt/nginx/sbin/nginx -c /opt/nginx/conf/nginx.conf -p /opt/nginx -t (code=exited, status=0/SUCCESS)
 Main PID: 10053 (nginx)
   CGroup: /system.slice/nginx.service
           ├─10053 nginx: master process /opt/nginx/sbin/nginx -c /opt/nginx/conf/nginx.conf...
           └─10054 nginx: worker process

1月 01 06:59:14 nginx systemd[1]: Starting Nginx Server...
1月 01 06:59:14 nginx nginx[10051]: nginx: the configuration file /opt/nginx/conf/nginx.... ok
1月 01 06:59:14 nginx nginx[10051]: nginx: configuration file /opt/nginx/conf/nginx.conf...ful
1月 01 06:59:14 nginx systemd[1]: Started Nginx Server.
Hint: Some lines were ellipsized, use -l to show in full.

[root@nginx ~]# netstat -ntlp | grep nginx
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      10053/nginx: master
```

## Nginx 配置

### 1.1 HTTP Load Balancing

+ 使用 NGINX 的 HTTP 模块使用 upstream 块在 HTTP 服务器上进行负载均衡

```conf
upstream backend {
    server 10.10.12.45:80       weight=1;
    server app.example.com:80   weight=2;
}
server {
    location / {
        proxy_pass http://backend;
    }
}
```

### 1.2 TCP Load Balancing
+ 使用 NGINX 的 stream 模块使用 upstream 块在 TCP 服务器上进行负载均衡

```conf
stream {
    upstream mysql_read {
        server read1.example.com:3306   weight=5;
        server read2.example.com:3306;
        server 10.10.12.34:3306         backup;
    }

    server {
        listen 3306;
        proxy_pass mysql_read;
    }
}
```

### 1.3 Load-Balancing 方法
+ 使用 NGINX 的负载平衡方法之一，例如least_conn, least_time, hash, ip_hash

这里将后端 upstream 池的负载平衡算法设置为最少连接。所有负载平衡算法（通用散列除外）都将是独立的指令，如前面的示例。通用散列采用单个参数（可以是变量的串联）来构建散列。

```conf
upstream backend {
    least_conn;
    server backend.example.com;
    server backend1.example.com;
}
```

并非所有请求或数据包都具有相同的权重。鉴于此，循环，甚至是先前示例中使用的加权循环，将不适合所有应用程序或交通流量的需要。NGINX提供了许多负载平衡算法，可用于适应特定的用例。 这些负载平衡算法或方法不仅可以选择，还可以配置。 以下负载平衡方法可用于 upstream HTTP，TCP 和 UDP 池：
1. Round robin（轮询）：  
默认负载平衡方法，按 upstream 池中服务器列表的顺序分配请求。对于加权循环，可以考虑权重，如果 upstream 服务器的容量变化，则可以使用加权循环。 权重的整数值越高，服务器在循环中的优势就越大。权重背后的算法只是加权平均的统计概率。循环法是默认的负载平衡算法，如果未指定其他算法，则使用该算法。
2. Least connections（）：  
另一种由 NGINX 提供的负载均衡方法。此方法通过将当前请求代理到具有通过 NGINX 代理的最少数量的打开连接的upstream服务器来平衡负载。在决定向哪个服务器发送连接时，最小连接（如循环）也会考虑权重。指令名称为 least_conn。
3. Least time（）：  
仅在 NGINX Plus 中可用，类似于最少连接，因为它代理具有最少数量的当前连接的upstream 服务器，但有利于具有最低平均响应时间的服务器。这种方法是最复杂的负载平衡算法之一，可满足高性能 Web 应用程序的需求。此算法是一个增加最少连接的值，因为少量连接并不一定意味着最快的响应。指令名称为 least_time。
4. Generic hash（）：  
管理员使用给定文本，请求或运行时的变量或两者来定义散列。NGINX 通过为当前请求生成哈希并将其放在 upstream 服务器上来分配服务器之间的负载。当您需要更多地控制发送请求的位置或确定哪些 upstream 服务器最有可能将数据缓存时，此方法非常有用。请注意，在池中添加或删除服务器时，将重新分配散列请求。该算法具有可选参数，一致，以最小化重新分布的影响。指令名称是 hash。
5. IP hash（）：  
仅支持HTTP，是最后一批。IP 哈希使用客户端 IP 地址作为哈希。与在通用散列中使用远程变量略有不同，此算法使用 IPv4 地址的前三个八位字节或整个 IPv6 地址。只要该服务器可用，此方法可确保客户端代理到同一个 upstream 服务器，这在会话状态受到关注且未由应用程序的共享内存处理时非常有用。 在分配散列时，此方法还会考虑权重参数。 指令名称为 ip_hash。

### 1.4 Connection Limiting
+ 使用 NGINX Plus 的 max_conns 参数限制 upstream 服务器的连接数

```conf
upstream backend {
    zone backends 64k;
    queue 750 timeout=30s;

    server webserver1.example.com max_conns=25;
    server webserver2.example.com max_conns=15;
}
```