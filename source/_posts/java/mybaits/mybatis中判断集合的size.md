

---
title: mybatis中判断集合的size
date: 2018-04-02 11:29:44
tags: [java,maven,mybatis]
categories: java
---



```xml
<if test="null != staffCodeList and staffCodeList.size > 0">
	and user_code  in
	<foreach item='item' index='index' collection="staffCodeList"  open="(" separator=","  close=")">
		#{item.code}
	</foreach>
</if>

```