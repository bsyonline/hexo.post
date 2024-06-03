---
title: python
tags: []
category:
  - python
author: bsyonline
lede: 没有摘要
date: 2018-05-31 23:20:09
thumbnail:
---



### 基本语法

#### 运算符
| 运算符                                                   | 说明            |
| ----------------------------------------------------- | ------------- |
| [] [:]                                                | 切片            |
| **                                                    | 指数            |
| ~                                                     | 按位取反          |
| ^                                                     | 按位异或          |
| \|                                                    | 按位或           |
| `+`， `-`， `*`， `/`， `%`， `//`                         | 加、减、乘、除、取模、整除 |
| >>                                                    | 右移            |
| <<                                                    | 左移            |
| `is`， `is not`                                        | 身份运算          |
| `in`， `not in`                                        | 成员运算          |
| `not`， `or`， `and`                                    | 逻辑运算          |
| `<=` ，`<` ，`>`， `>=`，`==`， `!=`                       | 比较运算          |
| `=`， `+=`， `-=`， `*=`， `/=`， `%=`， `//=`， `**=`， `&=` | 赋值运算          |

#### 数组


#### 字典


#### 切片


#### 模块
```python
def foo():  
    print('hello, world!')  
  
def foo():  
    print('hello, python!')
```
python 中一个文件就代表一个模块（module），如果同一个 .py 文件中有多个同名函数，只会有一个生效（最后一个）。不同的 module 中可以定义相同名称的函数。
`foo.py`
```python
def foo():  
    print('hello, world!')  
```
`bar.py`
```python
def foo():  
    print('hello, python!')  
```
`test.py`
```python
from foo import foo  
foo()  
from bar import foo  
foo()

import foo as f1  
import bar as f2  
f1.foo()  
f2.foo()
```
