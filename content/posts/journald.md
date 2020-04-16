---
title: journald 指南
date: 2019-12-31
lastmod: 2020-01-01
categories:
  - linux
tags:
  - systemd
  - journald
---

journald （systemd 日志管理服务）
<!--more-->

### 1. journald 服务

- systemd-journald 是一项收集和存储日志记录数据的系统服务。它基于从各种来源接收到的日志记录信息来创建和维护结构化, 已索引的日志：
  - 通过 kmsg 的内核日志消息
  - 通过 libc syslog 调用的简单系统日志消息
  - 通过本机的结构化系统日志消息日志API
  - 系统服务的标准输出和标准错误
  - 通过审核子系统的审核记录守护程序将以安全且可靠的方式隐式收集每个日志消息的大量元数据段

```bash
[root@nginx ~]# systemctl status systemd-journald.service
● systemd-journald.service - Journal Service
   Loaded: loaded (/usr/lib/systemd/system/systemd-journald.service; static; vendor preset: disabled)
   Active: active (running) since 三 2020-01-01 06:31:55 CST; 47min ago
     Docs: man:systemd-journald.service(8)
           man:journald.conf(5)
 Main PID: 515 (systemd-journal)
   Status: "Processing requests..."
   CGroup: /system.slice/systemd-journald.service
           └─515 /usr/lib/systemd/systemd-journald

1月 01 06:31:55 ldap.example.com systemd-journal[515]: Runtime journal is using 6.1M (max …M).
1月 01 06:31:55 ldap.example.com systemd-journal[515]: Journal started
1月 01 06:31:55 ldap.example.com systemd-journal[515]: Permanent journal is using 16.0M (m…G).
1月 01 06:31:55 ldap.example.com systemd-journal[515]: Time spent on flushing to /var is 7....
Hint: Some lines were ellipsized, use -l to show in full.
```

- systemd-journald.service 文件

```bash
[root@nginx ~]# cat /usr/lib/systemd/system/systemd-journald.service
#  This file is part of systemd.
#
#  systemd is free software; you can redistribute it and/or modify it
#  under the terms of the GNU Lesser General Public License as published by
#  the Free Software Foundation; either version 2.1 of the License, or
#  (at your option) any later version.

[Unit]
Description=Journal Service
Documentation=man:systemd-journald.service(8) man:journald.conf(5)
DefaultDependencies=no
Requires=systemd-journald.socket
After=systemd-journald.socket syslog.socket
Before=sysinit.target

[Service]
Type=notify
Sockets=systemd-journald.socket
ExecStart=/usr/lib/systemd/systemd-journald
Restart=always
RestartSec=0
StandardOutput=null
FileDescriptorStoreMax=4224
CapabilityBoundingSet=CAP_SYS_ADMIN CAP_DAC_OVERRIDE CAP_SYS_PTRACE CAP_SYSLOG CAP_AUDIT_CONTROL CAP_AUDIT_READ CAP_CHOWN CAP_DAC_READ_SEARCH CAP_FOWNER CAP_SETUID CAP_SETGID CAP_MAC_OVERRIDE
WatchdogSec=3min

# Increase the default a bit in order to allow many simultaneous
# services being run since we keep one fd open per service. Also, when
# flushing journal files to disk, we might need a lot of fds when many
# journal files are combined.
LimitNOFILE=16384
```

- 默认 journald 配置文件

```bash
[root@nginx ~]# cat /etc/systemd/journald.conf
#  This file is part of systemd.
#
#  systemd is free software; you can redistribute it and/or modify it
#  under the terms of the GNU Lesser General Public License as published by
#  the Free Software Foundation; either version 2.1 of the License, or
#  (at your option) any later version.
#
# Entries in this file show the compile time defaults.
# You can change settings by editing this file.
# Defaults can be restored by simply deleting this file.
#
# See journald.conf(5) for details.

[Journal]
#Storage=auto
#Compress=yes
#Seal=yes
#SplitMode=uid
#SyncIntervalSec=5m
#RateLimitInterval=30s
#RateLimitBurst=1000
#SystemMaxUse=
#SystemKeepFree=
#SystemMaxFileSize=
#RuntimeMaxUse=
#RuntimeKeepFree=
#RuntimeMaxFileSize=
#MaxRetentionSec=
#MaxFileSec=1month
#ForwardToSyslog=yes
#ForwardToKMsg=no
#ForwardToConsole=no
#ForwardToWall=yes
#TTYPath=/dev/console
#MaxLevelStore=debug
#MaxLevelSyslog=debug
#MaxLevelKMsg=notice
#MaxLevelConsole=info
#MaxLevelWall=emerg
#LineMax=48K
```

### 2. journald 日志持久化

默认情况下，日志将日志数据存储在 /run/log/journal/. 中。由于 /run/ 是易失性的，因此日志数据在重新引导时会丢失。要使数据持久化，只需创建 /var/log/journal/，然后systemd-journald将在其中存储数据：

```bash
[root@nginx ~]# mkdir -p /var/log/journal 
[root@nginx ~]# systemd-tmpfiles --create --prefix /var/log/journal
```

```bash
[root@nginx ~]# mkdir /etc/systemd/journald.conf.d
[root@nginx ~]# cat > /etc/systemd/journald.conf.d/99-prophet.conf <<EOF
[Journal]
# 持久化保存到磁盘
Storage=persistent

# 压缩历史日志
Compress=yes

SyncIntervalSec=5m
RateLimitInterval=30s
RateLimitBurst=1000

# 最大占用空间 10G
SystemMaxUse=10G

# 单日志文件最大 200M
SystemMaxFileSize=200M

# 日志保存时间 2 周
MaxRetentionSec=2week

# 不将日志转发到 syslog
ForwardToSyslog=no
EOF

[root@nginx ~]# systemctl restart systemd-journald
```


### 3. journalctl 管理工具

> journalctl 是用来查询由 systemd-journald 服务收集的 systemd 日志，当它不接收任何参数则会最早的收集入口展示日志

```bash
[root@nginx ~]# journalctl --help
journalctl [OPTIONS...] [MATCHES...]

Query the journal.

Flags:
     --system              展示系统日志
     --user                展示当前用户的用户日志
  -M --machine=CONTAINER   Operate on local container
  -S --since=DATE          展示从指定日期 [%Y-%m-%d %H:%M:%S] 开始的日志
  -U --until=DATE          展示到指定日期 [%Y-%m-%d %H:%M:%S] 为止的日志
  -c --cursor=CURSOR       Show entries starting at the specified cursor
     --after-cursor=CURSOR Show entries after the specified cursor
     --show-cursor         Print the cursor after all the entries
  -b --boot[=ID]           Show current boot or the specified boot
     --list-boots          Show terse information about recorded boots
  -k --dmesg               展示当前启动的内核日志
  -u --unit=UNIT           展示指定 unit [nginx, httpd ...] 的日志
  -t --identifier=STRING   Show entries with the specified syslog identifier
  -p --priority=RANGE      Show entries with the specified priority
  -e --pager-end           Immediately jump to the end in the pager
  -f --follow              Follow the journal
  -n --lines[=INTEGER]     Number of journal entries to show
     --no-tail             Show all lines, even in follow mode
  -r --reverse             Show the newest entries first
  -o --output=STRING       Change journal output mode (short, short-iso,
                                   short-precise, short-monotonic, verbose,
                                   export, json, json-pretty, json-sse, cat)
     --utc                 Express time in Coordinated Universal Time (UTC)
  -x --catalog             Add message explanations where available
     --no-full             Ellipsize fields
  -a --all                 Show all fields, including long and unprintable
  -q --quiet               Do not show privilege warning
     --no-pager            Do not pipe output into a pager
  -m --merge               Show entries from all available journals
  -D --directory=PATH      展示指定目录下的日志文件的日志
     --file=PATH           展示指定日志文件中的日志
     --root=ROOT           Operate on catalog files underneath the root ROOT
     --interval=TIME       Time interval for changing the FSS sealing key
     --verify-key=KEY      Specify FSS verification key
     --force               Override of the FSS key pair with --setup-keys

Commands:
  -h --help                Show this help text
     --version             Show package version
  -F --field=FIELD         List all values that a specified field takes
     --new-id128           Generate a new 128-bit ID
     --disk-usage          Show total disk usage of all journal files
     --vacuum-size=BYTES   Reduce disk usage below specified size
     --vacuum-time=TIME    Remove journal files older than specified date
     --flush               Flush all journal data from /run into /var
     --header              Show journal header information
     --list-catalog        Show all message IDs in the catalog
     --dump-catalog        Show entries in the message catalog
     --update-catalog      Update the message catalog database
     --setup-keys          Generate a new FSS key pair
     --verify              Verify journal file consistency
```

