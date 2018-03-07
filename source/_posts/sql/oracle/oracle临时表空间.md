
---
title: oracle临时表空间
date: 2018-02-01 14:29:44
tags: [oracle]
categories: 数据库
---


### 查询临时表空间的空闲程度：

```sql
select 'the ' || name || ' temp tablespaces ' || tablespace_name ||
       ' idle ' ||
       round(100 - (s.tot_used_blocks / s.total_blocks) * 100, 3) ||
       '% at ' || to_char(sysdate, 'yyyymmddhh24miss')
  from (select d.tablespace_name tablespace_name,
               nvl(sum(used_blocks), 0) tot_used_blocks,
               sum(blocks) total_blocks
          from v$sort_segment v, dba_temp_files d
         where d.tablespace_name = v.tablespace_name(+)
         group by d.tablespace_name) s,
       v$database;
```

### 有必要时创建一个新的临时表空间

```sql
create temporary tablespace TEMP04 TEMPFILE 
'D:\APP\ADMINISTRATOR\ORADATA\ORCL\TEMP04_01.DBF' SIZE 32767M 
REUSE AUTOEXTEND ON NEXT 128M MAXSIZE UNLIMITED;
```

### 有必要时增加临时表空间文件

```sql
ALTER TABLESPACE TEMP04 ADD TEMPFILE 'D:\APP\ADMINISTRATOR\ORADATA\ORCL\TEMP04_02.DBF' 
 SIZE 32767M  AUTOEXTEND ON NEXT 128M MAXSIZE unlimited;

```

### 有必要时更改默认临时表空间

```sql
alter database default temporary tablespace TEMP04; 
```

### 有必要时删除原有临时表空间

```sql
drop tablespace TEMP03 including contents and datafiles;  
```