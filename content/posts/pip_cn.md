---
title: pip 使用国内源
date: 2019-09-15 01:31:26
categories:
  - python
tags:
  - pip
  - python
---

pip 使用国内源
<!--more-->


```
mkdir ~/.pip
cat << EOF > ~/.pip/pip.conf
[global]
index-url = https://pypi.tuna.tsinghua.edu.cn/simple
[install]
trusted-host = tsinghua.edu.cn
EOF
```
