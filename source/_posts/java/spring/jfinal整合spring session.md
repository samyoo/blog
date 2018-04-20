---
title: jfinal整合spring session
date: 2018-03-05 11:29:44
tags: [java,spring]
categories: java
---

先添加依赖包
```xml
<!-- spring session -->
		<dependency>
			<groupId>org.springframework.session</groupId>
			<artifactId>spring-session</artifactId>
			<version>1.3.1.RELEASE</version>
		</dependency>
		<dependency>
			<groupId>org.springframework.session</groupId>
			<artifactId>spring-session-data-redis</artifactId>
			<version>1.3.1.RELEASE</version>
		</dependency>
		<!-- redis -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-redis</artifactId>
			<version>1.3.1.RELEASE</version>
		</dependency>

```

然后添加SpringSession拦截器

```java

/** spring session 拦截器
 * Created by sam on 4/9/18.
 */
public class SpringSessionInterceprot implements Interceptor {
    @Override
    public void intercept(Invocation inv) {

        Controller c = inv.getController();

        HttpSession session = c.getSession();

        String sessionId = c.getSession().getId();
       // String sessionId = "71f1e122-18d5-4e54-a019-0ba8c4397be8";


        //判断是否登录 ，没有登录 跳转页面
        if(!isLogin(sessionId,session)){
            c.redirect("/login");
            return;
        }

        inv.invoke();

    }


    /**
     * 判断是否登录
     * @param sessionId
     * @return
     */
    private boolean isLogin(String sessionId,HttpSession session){
        Map<Object,Object> map =  getSession(sessionId);

        //先判断session 是否过期
        if(!map.containsKey("maxInactiveInterval")) return false;

        Integer maxInactiveInterval = (Integer) map.get("maxInactiveInterval");

        if(maxInactiveInterval>0){

            UserVO vo = (UserVO)map.get("sessionAttr:LOGIN_USER");

            return true;

        }else{
            return false;
        }


    }

    /**
     * 在spring session里取值
     * @param sessionId
     * @return
     */
    private Map<Object,Object> getSession(String sessionId){
        String key = "spring:session:sessions:"+sessionId;
        Map<Object,Object> map = getRedis().boundHashOps(key).entries();
        return map;
    }

    /**
     *
     * @return
     */
    private RedisOperations<Object, Object> getRedis(){
        JedisShardInfo jsi = new JedisShardInfo(ConfigKit.getStr("session.redis.host"));
        JedisConnectionFactory jcf = new JedisConnectionFactory(jsi);
        return  sessionRedisTemplate(jcf);
    }

    private RedisTemplate<Object, Object> sessionRedisTemplate(
            RedisConnectionFactory connectionFactory) {
        RedisTemplate<Object, Object> template = new RedisTemplate<Object, Object>();
        template.setKeySerializer(new StringRedisSerializer());
        template.setHashKeySerializer(new StringRedisSerializer());
        template.setConnectionFactory(connectionFactory);
        template.afterPropertiesSet();
        return template;
    }
}

```