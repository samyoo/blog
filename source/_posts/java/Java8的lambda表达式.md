---
title: Java8的lambda表达式
date: 2018-02-27 10:29:44
tags: [java]
categories: java
---

### 为什么使用lambda表达式？

> Lambda表达式允许将函数作为方法的参数，或者将code作为数据。使得代码简洁，可读性较高。

### Lambda表达式的格式
> （attr1,attr2,.....）->函数体

左边是参数，当参数为1个时，可以没有括号，比如：`e->e+2 `  
当参数为0个时，用括号表示,比如:`     ()->System.out.println("表达式")`

如果函数体为一行，那么就没有必须显示的使用return语句，比如，下面的两个代码是等价的：
```java
Arrays.asList( "a", "b", "d" ).sort( ( e1, e2 ) -> e1.compareTo( e2 ) );
Arrays.asList( "a", "b", "d" ).sort( ( e1, e2 ) -> {
    int result = e1.compareTo( e2 );
    return result;
} );
```

如果函数体为多行，则需要将函数体用花括号”{ }”括起来，表达式用分号结尾。
```java
Arrays.asList( "a", "b", "d" ).sort( ( e1, e2 ) -> {
    int result = e1.compareTo( e2 );
    return result;
} );
```
