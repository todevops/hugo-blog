---
title: 将 Nginx 日志格式化成 json
date: 2019-11-19 17:27:38
tags:
---

将 Nginx 日志格式化成 json 格式
<!--more-->


```bash
vim /etc/nginx/nginx.conf
```


```conf
# 修改log_format
log_format log_json '{"timestamp":"$time_local",'
                                  '"http_host":"$http_host",'
                                  '"clinetip":"$remote_addr",'
                                  '"request":"$request",'
                                  '"status":"$status",'
                                  '"size":"$body_bytes_sent",'
                                  '"upstream_addr":"$upstream_addr",'
                                  '"upstream_status":"$upstream_status",'
                                  '"upstream_response_time":"$upstream_response_time",'
                                  '"request_time":"$request_time",'
                                  '"http_referer":"$http_referer",'
                                  '"http_user_agent":"$http_user_agent",'
                                  '"http_x_forwarded_for":"$http_x_forwarded_for"}';
access_log  /var/log/nginx/access.log  log_json;
```