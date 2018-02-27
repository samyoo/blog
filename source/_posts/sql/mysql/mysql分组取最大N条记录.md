---
title: mysql分组取最大N条记录
date: 2018-02-26 14:29:44
tags: [mysql]
categories: 数据库
---

### 先看原始数据
```sql
create table t2 (
    id int primary key,
    gid    char,
    col1    int,
    col2    int
) engine=innodb;

insert into t2 values (1,'A',31,6), (2,'B',25,83), (3,'C',76,21), (4,'D',63,56), (5,'E',3,17), (6,'A',29,97), (7,'B',88,63), (8,'C',16,22), (9,'D',25,43), (10,'E',45,28),
(11,'A',2,78), (12,'B',30,79), (13,'C',96,73), (14,'D',37,40), (15,'E',14,86), (16,'A',32,67), (17,'B',84,38), (18,'C',27,9), (19,'D',31,21), (20,'E',80,63),
(21,'A',89,9), (22,'B',15,22), (23,'C',46,84), (24,'D',54,79), (25,'E',85,64), (26,'A',87,13), (27,'B',40,45), (28,'C',34,90), (29,'D',63,8), (30,'E',66,40),
(31,'A',83,49), (32,'B',4,90), (33,'C',81,7), (34,'D',11,12), (35,'E',85,10), (36,'A',39,75), (37,'B',22,39), (38,'C',76,67), (39,'D',20,11), (40,'E',81,36);
```

### 期望结果
#### 1) N=1 取GID每组 COL2最大的记录
```
    +----+------+------+------+
    | id | gid  | col1 | col2 |
    +----+------+------+------+
    |  6 | A    |   29 |   97 |
    | 15 | E    |   14 |   86 |
    | 24 | D    |   54 |   79 |
    | 28 | C    |   34 |   90 |
    | 32 | B    |    4 |   90 |
    +----+------+------+------+
```

#### 2) N=3 取GID每组 COL2最大的3条记录

```
    +----+------+------+------+
    | id | gid  | col1 | col2 |
    +----+------+------+------+
    |  6 | A    |   29 |   97 |
    | 11 | A    |    2 |   78 |
    | 36 | A    |   39 |   75 |
    | 32 | B    |    4 |   90 |
    |  2 | B    |   25 |   83 |
    | 12 | B    |   30 |   79 |
    | 28 | C    |   34 |   90 |
    | 23 | C    |   46 |   84 |
    | 13 | C    |   96 |   73 |
    | 24 | D    |   54 |   79 |
    |  4 | D    |   63 |   56 |
    |  9 | D    |   25 |   43 |
    | 15 | E    |   14 |   86 |
    | 25 | E    |   85 |   64 |
    | 20 | E    |   80 |   63 |
    +----+------+------+------+
```

#### 第一题SQL写法

```sql
select a.* from t2 a  join (select gid,max(col2) col2 from t2 group by gid) b
on a.gid=b.gid and a.col2=b.col2;

SELECT a.id,a.gid,a.col1,a.col2 
FROM t2 as a,t2 as b 
where a.gid=b.gid AND a.col2 <=b.col2
GROUP BY a.id,a.gid,a.col1,a.col2 
having a.col2=max(b.col2)
ORDER BY a.gid,a.col2 desc
```
#### 第二题SQL写法

```sql
SELECT a.id,a.gid,a.col1,a.col2 
FROM t2  a,t2 b 
where a.gid=b.gid AND a.col2 <=b.col2
GROUP BY a.id,a.gid,a.col1,a.col2 
HAVING   COUNT(b.id) <=3 
ORDER BY a.gid,a.col2 desc
```