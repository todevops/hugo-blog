---
title: Linux 文本处理器之 awk
date: 2018-12-15
categories:
  - shell
tags:
  - shell
---

Linux 文本处理器之 awk
<!--more-->

```bash
awk  'BEGIN{FS="[:]"} {aa+=$3} END{print "所用的UID的和是:",aa}' passwd
FS	awk 'BEGIN{FS="[:]"} /root/{print $1,$2}' passwd
OFS	awk 'BEGIN{FS="[:]";OFS="---"} /root/{print $1,$2}' passwd
NF	awk -F: '{print NF}' passwd
RS	awk 'BEGIN{FS="[,]";RS=" "}{print $2}' bb.txt
ORS	awk 'BEGIN{FS="[:]";ORS="---"}{print $0}' passwd
FILENAME	awk -F"[:]" '{print FILENAME}' passwd
NR	awk '{print NR}' a.txt b.txt
FNR	awk '{print NR}' a.txt b.txt
# NR==FNR 此时处理的是第一个文件
# NR!=FNR 此时处理的是第二个文件
awk 'NR==FNR{print FILENAME} NR!=FNR{print FILENAME}' a.txt b.txt
if	awk -F[:] '{if($3<=1000){print $1"是root用户"}else{print $1"是普通用户"}}' /etc/passwd	
```


| | |
| - | - |
| 变量名    | 属性 |
| $0        | 当前记录 |
| $1~$n | 当前记录的第n个字段 |
| FS        | 输入字段分隔符，默认为空格 |
| RS        | 输入记录分割符，默认为换行符 |
| NF        | 当前记录中的字段个数，就是有多少列 |
| NR        | 已经读出的记录数，就是行号，从1开始 |
| OFS       | 输出字符分隔符，默认也是空格 |
| ORS       | 输出的记录分隔符，默认为换行符 |
