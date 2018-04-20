
---
title: oracle常用命令
date: 2018-04-20 14:29:44
tags: [oracle]
categories: 数据库
---

```sql

-- 查询所有用户的表与字段
SELECT * FROM all_tab_comments;
SELECT * FROM all_col_comments;

-- 查询当前用户的表与字段
SELECT * FROM user_tab_comments;
SELECT * FROM user_col_comments;

-- 查询当前用户所有表、序列
select * from user_objects ubs where ubs.OBJECT_TYPE='TABLE';
select * from user_objects ubs where ubs.OBJECT_TYPE='SEQUENCE';

-- 查询 DDL 语句
SELECT DBMS_METADATA.GET_DDL('TABLE','T_NAME') FROM DUAL;

-- 创建自增序列
create sequence ID
     minvalue 1
     maxvalue 99999999999999999999
     start with 1
     increment by 1
     nocache; 

-- SUBSTR 

```