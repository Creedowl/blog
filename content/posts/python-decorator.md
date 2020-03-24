---
title: decorator 装饰器
date: 2018-11-03T16:24:59+08:00
draft: false
toc: true
tags:
 - python
 - 学习笔记
---
## decorator 装饰器

![](https://ws4.sinaimg.cn/large/006tNbRwgy1fxlsbrfedaj30rs13ahdt.jpg)

### 普通装饰器

有个计算a+b的函数

```python
def add(a, b):
    return a+b
```

现在想要在函数执行前后进行操作，可以用装饰器

```python
from functools import wraps

def log(func):
    @wraps(func) # 避免原函数的__name__等属性发生变化
    def wrapper(*args, **kwargs)
        print('before func')
        func(*args, **kwargs) # 执行原函数，若不调用则不执行原函数
        print('after func')
    return wrapper
    
@log # @:装饰器语法糖
def add(a, b):
    return a+b
    
'''
add(1, 2)
output: 
before func
3
after func
'''
```

### 带参数的装饰器

要实现带参数的装饰器，只需要再用一个函数包装普通装饰器就行

```python
from functools import wraps

def log(text):
    def decorator(func):
        @wraps(func) # 避免原函数的__name__等属性发生变化
        def wrapper(*args, **kwargs)
            print(text)
            func(*args, **kwargs) # 执行原函数，若不调用则不执行原函数
            print('after func')
        return wrapper
    return decorator
    
@log('log') # @:装饰器语法糖
def add(a, b):
    return a+b
    
'''
add(1, 2)
output: 
log
3
after func
'''
```


