
---
title: Spring-Cloud整合Spring-Session的注意点
date: 2018-03-05 11:29:44
tags: [java,spring]
categories: java
---


zuul+feign+一个微服务的结构，然后想通过spring-session共享同一个session。通过简单的配置后，成功的运行了项目，但是发现zuul，feign，微服务中三处打印出来的sessionId都不相同。通过浏览器调试可以确定spring-session是其作用了。

如果是tomcat的session，cookie的name应该是JSESSIONID，这里显示SESSION说明spring-session是其作用了，而且也可以在redis中找到对应的记录。那么产生三处session都不同的原因应该就是zuul转发给feign，feign调用微服务的时候都没有把cookie传过去。找到原因后，就有解决的思路了。

通过查找spring-cloud的文档发现，zuul默认是屏蔽Cookie的，要想使用Cookie，要在application.yml中配置sensitiveHeaders并把它设为空，这是一个屏蔽的黑名单，默认不为空，会屏蔽Cookie，例子如下:

```
zuul:
  routes:
    feign-a:
      path: /feign-a/**
      serviceId: fegin-service-a
      sensitiveHeaders:
```

通过这个配置zuul已经可以把cookie传到feign中了，那么feign要如何把cookie传到微服务中呢?我们可以通过feign的RequestInterceptor接口来实现

```
@Configuration
public class FeignConfig {

    @Bean
    public RequestInterceptor requestInterceptor() {
        return requestTemplate -> {
            String sessionId = RequestContextHolder.currentRequestAttributes().getSessionId();
            if (!Strings.isNullOrEmpty(sessionId)) {
                requestTemplate.header("Cookie", "SESSION=" + sessionId);
            }
        };
    }
}

```

最后还是出错了，报了找不到与线程绑定的request的异常。通过调试初步判断应该是feign整合了hystrix的锅。通过更改hystrix的默认策略为SEMAPHORE可以解决。

```
hystrix:
  command:
    default:
      execution:
        isolation:
          strategy: SEMAPHORE
```
