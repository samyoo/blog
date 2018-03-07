
---
title: sqlserver常用命令
date: 2018-02-25 14:29:44
tags: [sqlserver]
categories: 数据库
---


```sql
-- 显示所有数据库
SELECT name FROM master.dbo.sysdatabases;

-- 显示所有表名
select name from sys.objects where type='U';
-- 或者
select name from sys.tables;
-- 或者
SELECT table_name, table_type FROM information_schema.tables;

-- 显示所有字段
select * from syscolumns where id=object_id('TABLES_NAME')

-- 显示字段信息
exec sp_columns TABLES_NAME;

-- 修改字段名称
exec sp_rename 'TABLES_NAME.oldName','newName';

-- 显示所以索引
exec sp_helpindex  TABLES_NAME;
exec sp_help TABLES_NAME;

-- 种子值
SELECT IDENT_CURRENT('TABLES_NAME');
DBCC CHECKIDENT(TABLES_NAME, RESEED, 1);

-- 查看表结构
SELECT (case when a.colorder=1 then d.name else null end) 表名,  
a.colorder 字段序号,a.name 字段名,
(case when COLUMNPROPERTY( a.id,a.name,'IsIdentity')=1 then '√'else '' end) 标识, 
(case when (SELECT count(*) FROM sysobjects  
WHERE (name in (SELECT name FROM sysindexes  
WHERE (id = a.id) AND (indid in  
(SELECT indid FROM sysindexkeys  
WHERE (id = a.id) AND (colid in  
(SELECT colid FROM syscolumns WHERE (id = a.id) AND (name = a.name)))))))  
AND (xtype = 'PK'))>0 then '√' else '' end) 主键,b.name 类型,a.length 占用字节数,  
COLUMNPROPERTY(a.id,a.name,'PRECISION') as 长度,  
isnull(COLUMNPROPERTY(a.id,a.name,'Scale'),0) as 小数位数,(case when a.isnullable=1 then '√'else '' end) 允许空,  
isnull(e.text,'') 默认值,isnull(g.[value], ' ') AS [说明]
FROM  syscolumns a 
left join systypes b on a.xtype=b.xusertype  
inner join sysobjects d on a.id=d.id and d.xtype='U' and d.name<>'dtproperties' 
left join syscomments e on a.cdefault=e.id  
left join sys.extended_properties g on a.id=g.major_id AND a.colid=g.minor_id
left join sys.extended_properties f on d.id=f.class and f.minor_id=0
where  d.name='TABLES_NAME' 
order by a.id,a.colorder
```