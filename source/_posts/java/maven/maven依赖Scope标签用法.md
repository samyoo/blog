---
title: maven依赖Scope标签用法
date: 2018-03-11 14:29:44
tags: [java,maven]
categories: java
---


### 在一个maven项目中，如果存在编译需要而发布不需要的jar包，可以用scope标签，值设为provided。如下：

```xml
        <dependency>
            <groupId>javax.servlet.jsp</groupId>
            <artifactId>jsp-api</artifactId>
            <version>2.1</version>
            <scope>provided</scope>
            <classifier />
        </dependency>
```

### scope的其他参数如下：

#### compile
默认的scope，表示 dependency 都可以在生命周期中使用。而且，这些dependencies 会传递到依赖的项目中。适用于所有阶段，会随着项目一起发布

#### provided
跟compile相似，但是表明了dependency 由JDK或者容器提供，例如Servlet AP和一些Java EE APIs。这个scope 只能作用在编译和测试时，同时没有传递性。????????

#### runtime
表示dependency不作用在编译时，但会作用在运行和测试时，如JDBC驱动，适用运行和测试阶段。

#### test
表示dependency作用在测试时，不作用在运行时。 只在测试时使用，用于编译和运行测试代码。不会随项目发布。

#### system
跟provided 相似，但是在系统中要以外部JAR包的形式提供，maven不会在repository查找它。
