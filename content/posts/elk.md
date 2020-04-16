---
title: 使用 ELK 搭建日志分析系统
date: 2019-11-19 13:22:39
tags:
---

使用 ELK 搭建日志分析系统
<!--more-->


```
cat << EOF >> /etc/security/limits.conf
* soft nproc 65535
* hard nproc 65535
* soft nofile 65535
* hard nofile 65535
EOF
```


```conf
input {
    tcp {
        port => 8002
        codec => json_lines
    }
}

filter{
	# 如果message中以Retrieved hosts from InstanceDiscovery: 0开头
    if([message]=~ "^Retrieved hosts from InstanceDiscovery: 0"){
    	# 丢弃
        drop{}
    }
}

output {
        elasticsearch {
                hosts => "localhost:8001"
        }
        stdout { codec => rubydebug}
}
```

```conf
filter {
    json {
        source => "message"
    }
}
```