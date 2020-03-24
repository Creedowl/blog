---
title: var和let声明变量的区别
date: 2019-01-26T21:59:14+08:00
draft: false
toc: true
tags:
 - js
---
# var和let声明变量的区别

> 今天因为全局变量重复声明的问题被let坑了好久，就研究了一下var和let的区别

## var

var是最早的变量标识符（但在js中声明变量也不一定要标识符。。），兼容性也是最好的，但也存在一些问题，如

1. 变量提升，即函数和变量的声明都将被提升到函数的最顶部。但初始化并不会提升

```javascript
{
    // 变量a未声明
    console.log(a) // error: Uncaught ReferenceError: a is not defined
}

{
    // 变量声明提升，但未初始化
    console.log(a) // undefined
    var a = 2 // 初始化a
    console.log(a) // 2
}
```

2. 可重复声明

```javascript
var a = 2
var a = '3'
```

3. 没有局部变量，变量作用域都是全局容易导致

## let

为了解决var的一些奇奇怪怪的问题，es6新增了let命令用于声明变量。与var相比，let更像其他语言（如c艹）

1. 不存在变量提升，所以类似上面的写法会抛出未定义的错误

```javascript
{
    console.log(a) // error: Uncaught ReferenceError: a is not defined
    let a = 2 // 初始化a
    console.log(a) // 2
}
```

2. 不可重复声明

```javascript
let a = 1
let a = 2 // SyntaxError: Identifier 'a' has already been declared
```

3. 有块级作用域等特性，所以如循环变量等局部变量等不会影响外部的变量