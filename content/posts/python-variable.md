---
title: Python 判断变量的类型
date: 2019-11-20 18:10:52
tags:
  - python
---

Python 判断变量的类型的方法
<!--more-->

## python 判断变量的类型

python的数据类型有：
- 数字(int)
- 浮点(float)
- 字符串(str)
- 列表(list)
- 元组(tuple)
- 字典(dict)
- 集合(set)

一般通过以下方法判断变量类型：
- isinstance(参数1, 参数2)

```python
i = 1
isinstance(i, int)
True
```

```python
i = "test"
isinstance(i, str)
True
```

```python
i = (1, 2, 3, 4, 5)
isinstance(i, tuple)
True
```

- type(参数1)

```python
i = 1
type(i)
<class 'int'>
```

```python
i = "test"
type(i)
<class 'str'>
```

```python
i = (1, 2, 3, 4, 5)
type(i)
<class 'tuple'>
```

## python 变量转换错误 ValueError

```python
i = 3.1415926
int(i)
3
```

```python
i = 2
float(i)
2.0
```

```python
i = "test"
int(i)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ValueError: invalid literal for int() with base 10: 'test'
```

```python
i = "test"
tuple(i)
('t', 'e', 's', 't')
```

```python
i = "test"
list(i)
['t', 'e', 's', 't']
```

```python
i = "test"
dict(i)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ValueError: dictionary update sequence element #0 has length 1; 2 is required
```

```python
i = "test"
set(i)
{'e', 's', 't'}
```