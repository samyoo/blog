
---
title: oracle备份还原
date: 2018-02-02 14:29:44
tags: [oracle]
categories: 数据库
---

### 查看表结构
```sql
select dbms_metadata.get_ddl('TABLE','TABLE_NAME') from dual;

DBMS_METADATA.GET_DDL (
object_type     IN VARCHAR2,
name            IN VARCHAR2,
schema          IN VARCHAR2 DEFAULT NULL,
version         IN VARCHAR2 DEFAULT 'COMPATIBLE',
model           IN VARCHAR2 DEFAULT 'ORACLE',
transform       IN VARCHAR2 DEFAULT 'DDL')
RETURN CLOB;

SELECT DBMS_METADATA.GET_DDL('TABLE','EMP','SCOTT')
FROM DUAL;

SELECT DBMS_METADATA.GET_DDL('INDEX','IDX_EMP','SCOTT')
FROM DUAL;

```

### 备份
```sql
declare
          h1   NUMBER;
          l1   varchar2(20);
         begin
             h1 := dbms_datapump.open (operation => 'EXPORT', job_mode => 'SCHEMA', job_name => 'JOB_EXP1', version => 'COMPATIBLE'); 
           dbms_datapump.set_parallel(handle => h1, degree => 1); 
           dbms_datapump.add_file(handle => h1, filename => 'EXPDAT.LOG', directory => 'DATA_PUMP_DIR', filetype => 3); 
           dbms_datapump.set_parameter(handle => h1, name => 'KEEP_MASTER', value => 0); 
           dbms_datapump.metadata_filter(handle => h1, name => 'SCHEMA_EXPR', value => 'IN(''sam_300'')'); 
           dbms_datapump.add_file(handle => h1, filename => 'sam_300_DB.DMP', directory => 'DATA_PUMP_DIR', filetype => 1); 
           dbms_datapump.set_parameter(handle => h1, name => 'INCLUDE_METADATA', value => 1); 
           dbms_datapump.set_parameter(handle => h1, name => 'DATA_ACCESS_METHOD', value => 'AUTOMATIC'); 
           dbms_datapump.set_parameter(handle => h1, name => 'ESTIMATE', value => 'BLOCKS'); 
           dbms_datapump.start_job(handle => h1, skip_current => 0, abort_step => 0); 
           dbms_datapump.wait_for_job(handle => h1, job_state => l1);
           dbms_datapump.detach(handle => h1); 
        end;


-- 2、注意：
A、 'IN(''CHY'')' 中的CHY为PL\SQL登陆的用户名，待备份的用户，注意用大写。
B、filename => 'CHYDB.DMP'中的CHYDB是指定的备份出的dmp文件名称，注意用大写。
-- 3、待plsql中执行完成，从下面查询获取备份文件的路径，并将dmp文件可以拷贝出来
SELECT directory_path FROM dba_directories WHERE directory_name='DATA_PUMP_DIR';

```

### 还原

#### 在新装好的oracle里，用plsql以system登录先创建用户

```sql
/*创建用户分为四步 */
/*第1步：创建临时表空间  */
create temporary tablespace user_temp  
tempfile 'D:\orcldata\user_temp.dbf' 
size 50m  
autoextend on  
next 50m maxsize 20480m  
extent management local;  
 
/*第2步：创建数据表空间  */
create tablespace user_data  
logging  
datafile 'D:\orcldata\user_data.dbf' 
size 50m  
autoextend on  
next 50m maxsize 20480m  
extent management local;  
 
/*第3步：创建用户并指定表空间  */
create user sam_300 identified by sam123  
default tablespace user_data  
temporary tablespace user_temp;  
 
/*第4步：给用户授予权限  */
grant connect,resource,dba to sam_300;
```

#### 、然后创建目录别名

```sql
create or replace directory orabak  as  'd://orclbak';
```

#### 打开CMD命令程序运行,由于安装的服务器系统是中文的所以要设置字符集

```sql
set NLS_LANG=AMERICAN_AMERICA.UTF8

/*修改dmp文件名和日志名（备份要在d://orclbak）*/
/*语句二选一*/
/*用户名都是sam_300*/
impdp system/sam123 directory=orabak dumpfile=sam_300_EXPDP_2017-05-14_BAK.DMP logfile=impdp_sam_300_EXPDP_2017-05-14.log schemas=sam_300 transform=oid:n
/*用户名是其他，还要修改新用户名*/
impdp system/sam123 directory=orabak dumpfile=sam_300_EXPDP_2017-05-14_BAK.DMP logfile=impdp_sam_300_EXPDP_2017-05-14.log remap_schema=sam_300:sam_300_2 transform=oid:n
```