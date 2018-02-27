---
title: java8的一些使用示例
date: 2018-02-28 14:29:44
tags: [java]
categories: java
---

#### wdt的签名sign算法
```java
    private static String getSign(Map<String,String> paramMap){
        return MD5Util.getMD5(packData(paramMap)+appsecret);
    }

    private static String packData(Map<String,String> paramMap){
        return paramMap.keySet().stream().sorted()
                .map(key->getKey(key)+":"+getVal(paramMap.get(key)))
                .collect(Collectors.joining(";"));
    }
```

#### 按名称分组
```java
      Map<String, List<Record>> map =  list.stream()
           .collect(Collectors.groupingBy(record->record.getStr("name")));

```

#### List To Map
```java
 // key = name, value - id
 Map<String, Long> result2 = list.stream().collect(
         Collectors.toMap(Bean::getName, Bean::getId));

 // Same with result1, just different syntax
 // key = id, value = name
 Map<Integer, String> result3 = list.stream().collect(
          Collectors.toMap(x -> x.getId(), x -> x.getName()));

```


#### 返回当前日期，在一年中的第几周
```java
    public static int getLocalWeek(LocalDate date){
        // Or use a specific locale, or configure your own rules
        //WeekFields weekFields = WeekFields.of(Locale.CHINESE);
        int weekNumber = date.get(WeekFields.ISO.weekOfWeekBasedYear());
        return weekNumber;
    }
```

#### 格式化日期时间
```java
   public static String format(LocalDateTime ldt){
        DateTimeFormatter dt = new DateTimeFormatterBuilder()
                .appendValue(HOUR_OF_DAY, 2)
                .appendLiteral(':')
                .appendValue(MINUTE_OF_HOUR, 2)
                .optionalStart()
                .appendLiteral(':')
                .appendValue(SECOND_OF_MINUTE, 2).toFormatter();

        DateTimeFormatter df = new DateTimeFormatterBuilder()
                .parseCaseInsensitive()
                .append(DateTimeFormatter.ISO_LOCAL_DATE)
                .appendLiteral(' ')
                .append(dt).toFormatter();

        return ldt.format(df);
    }
```
