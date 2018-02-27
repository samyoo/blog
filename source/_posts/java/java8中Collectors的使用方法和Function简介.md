---
title: java8中Collectors的使用方法和Function简介
date: 2018-02-28 9:29:44
tags: [java]
categories: java
---

### Collectors 实现了接口 `Collector<T,A,R>`

```
T:需要进行reduce操作的元素类型 
A:reduce操作的动态集合类型 
R:reduce操作的结果类型
```

### 举例
```java
//将名字集合到list
List<String> list = people.stream().map(Person::getName).collect(Collectors.toList());


//将名字集合到TreeSet
Set<String> set = people.stream().map(Person::getName)
                 .collect(Collectors.toCollection(TreeSet::new));

//将名字转换为String,并连接为一个字符串
String joined = things.stream()
                      .map(Object::toString)
                      .collect(Collectors.joining(", "));

//计算员工的工资总额
int total = employees.stream()
                     .collect(Collectors.summingInt(Employee::getSalary)));

//根据部门将员工分组：
Map<Department, List<Employee>> byDept = employees.stream()
                    .collect(Collectors.groupingBy(Employee::getDepartment));

//计算不同部门的员工工资总额
Map<Department, Integer> totalByDept= employees.stream()
                       .collect(Collectors.groupingBy(
                        Employee::getDepartment, 
                        Collectors.summingInt(Employee::getSalary)));

//划分及格和不及格的学生
Map<Boolean, List<Student>> passingFailing = students.stream()
                 .collect(Collectors.partitioningBy(
                         s -> s.getGrade() >= PASS_THRESHOLD));


```

### `Function<T, R>`
```
T:函数的输入类型 
R:函数的输出类型
```
也就是通过这个函数，可以将一个类型转换为另一个类型，比如下面的例子：
```java
 //定义一个function 输入是String类型，输出是 EventInfo 类型，  EventInfo是一个类。           
Function<String, EventInfo> times2 = fun -> { 
              EventInfo a = new EventInfo(); 
              a.setName(fun); 
              return a;
};

String[] testintStrings={"1","2","3","4"}; 

//将String 的Array转换成map,调用times2函数进行转换
Map<String,EventInfo> eventmap1=Stream.of(testintStrings)
                        .collect(Collectors.toMap(
                         inputvalue->inputvalue, 
                         inputvalue->times2.apply(inputvalue)));
```

如果`Collectors.toMap`的转换过程很简单，比如输入和输出类型相同，则不需要另外定义`Function`,例如：
```java
Map<String,String> eventmap2=Stream.of(testStrings)
                  .collect(Collectors.toMap(
                   inputvalue->inputvalue, 
                   inputvalue->(inputvalue+"a")));
```

### Java中有哪些函数式接口呢？
```
java.lang.Runnable 
java.util.concurrent.Callable 
java.security.PrivilegedAction 
java.util.Comparator 
java.io.FileFilter 
java.nio.file.PathMatcher 
java.lang.reflect.InvocationHandler 
java.beans.PropertyChangeListener 
java.awt.event.ActionListener 
javax.swing.event.ChangeListener
```
```
java.util.function — 中定义了几组类型的函数式接口以及针对基本数据类型的子类。 
Predicate — 传入一个参数，返回一个bool结果， 方法为boolean test(T t) 
Consumer — 传入一个参数，无返回值，纯消费。 方法为void accept(T t) 
Function — 传入一个参数，返回一个结果，方法为R apply(T t) 
Supplier — 无参数传入，返回一个结果，方法为T get() 
UnaryOperator — 一元操作符， 继承Function,传入参数的类型和返回类型相同。 
BinaryOperator — 二元操作符， 传入的两个参数的类型和返回类型相同， 继承BiFunction 
```