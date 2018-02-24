---
title: mysql递归查询
date: 2018-02-23 14:29:44
tags: mysql
categories: mysql
---

### 样例数据：
```sql
create table treeNodes(
   id int primary key,
   nodename varchar(20),
   pid int 
);

select * from treeNodes;
+----+----------+------+
| id | nodename | pid  |
+----+----------+------+
|  1 | A        |    0 |
|  2 | B        |    1 |
|  3 | C        |    1 |
|  4 | D        |    2 |
|  5 | E        |    2 |
|  6 | F        |    3 |
|  7 | G        |    6 |
|  8 | H        |    0 |
|  9 | I        |    8 |
| 10 | J        |    8 |
| 11 | K        |    8 |
| 12 | L        |    9 |
| 13 | M        |    9 |
| 14 | N        |   12 |
| 15 | O        |   12 |
| 16 | P        |   15 |
| 17 | Q        |   15 |
+----+----------+------+
17 rows in set (0.00 sec)
```

### 树形图如下

```
 1:A
  +-- 2:B
  |    +-- 4:D
  |    +-- 5:E
  +-- 3:C
       +-- 6:F
            +-- 7:G
 8:H
  +-- 9:I
  |    +-- 12:L
  |    |    +--14:N
  |    |    +--15:O
  |    |        +--16:P
  |    |        +--17:Q
  |    +-- 13:M
  +-- 10:J
  +-- 11:K  
```

创建一个function getChildLst, 得到一个由所有子节点号组成的字符串. 

```sql
 delimiter //
 CREATE FUNCTION `getChildLst`(rootId INT)
     RETURNS varchar(1000)
     BEGIN
       DECLARE sTemp VARCHAR(1000);
       DECLARE sTempChd VARCHAR(1000);
    
       SET sTemp = '$';
       SET sTempChd =cast(rootId as CHAR);
    
       WHILE sTempChd is not null DO
         SET sTemp = concat(sTemp,',',sTempChd);
         SELECT group_concat(id) INTO sTempChd FROM treeNodes where FIND_IN_SET(pid,sTempChd)>0;
       END WHILE;
       RETURN sTemp;
     END
     //
 delimiter ;
```

使用我们直接利用find_in_set函数配合这个getChildlst来查找

```
select getChildLst(1);
+-----------------+
| getChildLst(1)  |
+-----------------+
| $,1,2,3,4,5,6,7 |
+-----------------+

select * from treeNodes where FIND_IN_SET(id, getChildLst(1));
+----+----------+------+
| id | nodename | pid  |
+----+----------+------+
|  1 | A        |    0 |
|  2 | B        |    1 |
|  3 | C        |    1 |
|  4 | D        |    2 |
|  5 | E        |    2 |
|  6 | F        |    3 |
|  7 | G        |    6 |
+----+----------+------+

select * from treeNodes where FIND_IN_SET(id, getChildLst(3));
+----+----------+------+
| id | nodename | pid  |
+----+----------+------+
|  3 | C        |    1 |
|  6 | F        |    3 |
|  7 | G        |    6 |
+----+----------+------+
```

 

>优点：简单，方便，没有递归调用层次深度的限制 (max_sp_recursion_depth,最大255);

>缺点：长度受限，虽然可以扩大 RETURNS varchar(1000)，但总是有最大限制的。


