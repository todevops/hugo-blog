---
title: 开启mysql慢查询和死锁日志
date: 2019-09-15 01:31:26
categories: ["database"]
tags:
  - mysql
---

开启mysql慢查询和死锁日志
<!--more-->

### 显示慢查询状态及日志目录

```
show variables like '%slow_query_log%';

+---------------------+--------------------------------------+
| Variable_name       | Value                                |
+---------------------+--------------------------------------+
| slow_query_log      | OFF                                  |
| slow_query_log_file | /var/lib/mysql/9e6bec7d7160-slow.log |
+---------------------+--------------------------------------+
```

开启慢查询

说明: 1开启;0关闭;

```
set global slow_query_log = 1;
```

显示慢查询阈值(单位秒)

默认执行时间超过10s才会被记录到日志

```
show variables like '%long_query%';

+-----------------+-----------+
| Variable_name   | Value     |
+-----------------+-----------+
| long_query_time | 10.000000 |
+-----------------+-----------+
```

设置慢查询阈值

注意:设置后需要重新打开mysql客户端才能到最新的值

```
set global long_query_time = 0.8;
```

查看死锁的日志是否开启

```
show variables like "%innodb_print_all_deadlocks%";

+----------------------------+-------+
| Variable_name              | Value |
+----------------------------+-------+
| innodb_print_all_deadlocks | OFF   |
+----------------------------+-------+
```

开启记录死锁

```
set global innodb_print_all_deadlocks=1
```

查看最近一次deadlock

```
show engine innodb status\G;
```
