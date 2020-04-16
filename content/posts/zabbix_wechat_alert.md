---
title: Zabbix 企业微信告警
date: 2018-11-14
categories:
  - linux
tags:
  - zabbix
  - python
---

使用 python 编写 Zabbix 企业微信告警脚本
<!--more-->

## 1. 编辑 zabbix_server.conf 配置 zabbix 告警脚本路径
```
AlertScriptsPath=/usr/local/share/zabbix/alertscripts/
```

## 2. 创建发送消息脚本
+ 编写脚本 vim wechat.py

```python
#!/usr/bin/env python
#coding=utf-8
import requests
import json
import os
import sys
# 基本信息
CropID = 'xxxxxxxxxx'
Secret = 'xxxxxxxxxx'
agentid = 'xxxxxxxxx'
touser = 'xxxxxxxxxx' 
# 获取Token
GetToken ="https://qyapi.weixin.qq.com/cgi-bin/gettoken?corpid="+ CropID + "&corpsecret=" + Secret
headers = {'Content-Type': 'application/json'}
json_data = json.loads(requests.get(GetToken).content.decode())
token = json_data["access_token"]
# 消息发送接口
Purl = "https://qyapi.weixin.qq.com/cgi-bin/message/send?access_token=" + token
# 消息发送函数
def sendmsg(message):
    weixin_msg = {
        "touser" : "TangYingJie",         
        "msgtype" : "text",
        "agentid" : 1000002,
        "text" : {
            "content" : message
         },
     }
    print requests.post(Purl,json.dumps(weixin_msg),headers=headers)
  
if __name__ == '__main__':
    message = sys.argv[1]      #获取第二个参数
    sendmsg(message)
```

+ 测试脚本发送消息

```
./wechat.py 测试消息
```
## 3. 进入 zabbix 主界面配置
### 报警媒介类型
![]({{ "/assets/images/zabbix_alert/1.png" | absolute_url }})

### 用户 -> 报警媒介
![]({{ "/assets/images/zabbix_alert/2.png" | absolute_url }})

### 动作 -> 操作
![]({{ "/assets/images/zabbix_alert/3.png" | absolute_url }})

+ 默认接收人

```
{TRIGGER.STATUS} : {TRIGGER.NAME}
```
+ 默认信息

```
当前状态 : {TRIGGER.STATUS}
告警主机 : {HOST.NAME}
告警地址 : {HOST.IP}
告警时间 : {EVENT.DATE} {EVENT.TIME}
告警等级 : {TRIGGER.SEVERITY}
告警信息 : {TRIGGER.NAME}
监控取值 : {ITEM.VALUE}
监控项目 : {ITEM.NAME}
持续时间 : {EVENT.AGE}
事件ID : {ITEM.ID}
```
### 动作 -> 恢复操作
![]({{ "/assets/images/zabbix_alert/4.png" | absolute_url }})