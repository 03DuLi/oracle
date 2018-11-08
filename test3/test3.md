# 实验3：创建分区表 
## 实验目的：
掌握分区表的创建方法，掌握各种分区方式的使用场景。
## 实验内容：
- 本实验使用3个表空间：USERS，USERS02，USERS03在表空间中创建两张表：订单表（订单）与订单详表（ORDER_DETAILS）。
- 使用**你自己的账号**创建本实验的表，表创建在上述3个分区，自定义分区策略。
- 你需要使用系统用户给你自己的账号分配上述分区的使用权限你需要使用系统用户给你的用户分配可以查询执行计划的权限。
- 表创建成功后，插入数据，数据能并平均分布到各个分区每个表的数据都应该大于1万行，对表进行联合查询。
- 写出插入数据的语句和查询数据的语句，并分析语句的执行计划。
-进行分区与不分区的对比实验。
### 用户名：du_li
## 实验步骤
在主表orders和从表order_details之间建立引用分区
在study用户中创建两个表：orders（订单表）和order_details（订单详表），两个表通过列order_id建立主外键关联。orders表按范围分区进行存储，order_details使用引用分区进行存储。
### 创建user02,user03的表空间:
```sql
SQL > create tablespace users02 datafile' / home / oracle / app / oracle / oradata / orcl / pdborcl / pdbtest_users02_1。DBF 'SIZE 100M AUTOEXTEND ON NEXT 50M MAXSIZE UNLIMITED，
' / home / oracle / app / oracle / oradata / orcl / pdborcl / pdbtest_users02_2。DBF 'SIZE 100M AUTOEXTEND ON NEXT 50M MAXSIZE UNLIMITED，
EXTENT MANAGEMENT本地区域空间管理自动;
表空间已创建
SQL > create tablespace users03 datafile' / home / oracle / app / oracle / oradata / orcl / pdborcl / pdbtest_users03_1。DBF 'SIZE 100M AUTOEXTEND ON NEXT 50M MAXSIZE UNLIMITED，
' / home / oracle / app / oracle / oradata / orcl / pdborcl / pdbtest_users03_2。DBF 'SIZE 100M AUTOEXTEND ON NEXT 50M MAXSIZE UNLIMITED，
EXTENT MANAGEMENT本地区域空间管理自动;
表空间已创建
```
### 给自己的账号分配分区的使用权限:
```sql
SQL> ALTER USER du_li QUOTA 50M ON users02;
User altered.
SQL> ALTER USER du_li QUOTA 50M ON users03;
User altered
```
### 创建orders表的语句是：
```sql
CREATE TABLE ORDERS 
(
  ORDER_ID NUMBER(10, 0) NOT NULL 
, CUSTOMER_NAME VARCHAR2(40 BYTE) NOT NULL 
, CUSTOMER_TEL VARCHAR2(40 BYTE) NOT NULL 
, ORDER_DATE DATE NOT NULL 
, EMPLOYEE_ID NUMBER(6, 0) NOT NULL 
, DISCOUNT NUMBER(8, 2) DEFAULT 0 
, TRADE_RECEIVABLE NUMBER(8, 2) DEFAULT 0 
, CONSTRAINT ORDERS_PK PRIMARY KEY 
  (
    ORDER_ID 
  )
  USING INDEX 
  (
      CREATE UNIQUE INDEX ORDERS_PK ON ORDERS (ORDER_ID ASC) 
      LOGGING 
      TABLESPACE USERS 
      PCTFREE 10 
      INITRANS 2 
      STORAGE 
      ( 
        BUFFER_POOL DEFAULT 
      ) 
      NOPARALLEL 
  )
  ENABLE 
) 
TABLESPACE USERS 
PCTFREE 10 
INITRANS 1 
STORAGE 
( 
  BUFFER_POOL DEFAULT 
) 
NOCOMPRESS 
NOPARALLEL 
PARTITION BY RANGE (ORDER_DATE) 
(
  PARTITION PARTITION_BEFORE_2016 VALUES LESS THAN (TO_DATE(' 2016-01-01 00:00:00', 'SYYYY-MM-DD HH24:MI:SS', 'NLS_CALENDAR=GREGORIAN')) 
  NOLOGGING 
  TABLESPACE USERS 
  PCTFREE 10 
  INITRANS 1 
  STORAGE 
  ( 
    INITIAL 8388608 
    NEXT 1048576 
    MINEXTENTS 1 
    MAXEXTENTS UNLIMITED 
    BUFFER_POOL DEFAULT 
  ) 
  NOCOMPRESS NO INMEMORY  
, PARTITION PARTITION_BEFORE_2017 VALUES LESS THAN (TO_DATE(' 2017-01-01 00:00:00', 'SYYYY-MM-DD HH24:MI:SS', 'NLS_CALENDAR=GREGORIAN')) 
  NOLOGGING 
  TABLESPACE USERS02 
  PCTFREE 10 
  INITRANS 1 
  STORAGE 
  ( 
    INITIAL 8388608 
    NEXT 1048576 
    MINEXTENTS 1 
    MAXEXTENTS UNLIMITED 
    BUFFER_POOL DEFAULT 
  ) 
  NOCOMPRESS NO INMEMORY  
, PARTITION PARTITION_BEFORE_2018 VALUES LESS THAN (TO_DATE(' 2018-01-01 00:00:00', 'SYYYY-MM-DD HH24:MI:SS', 'NLS_CALENDAR=GREGORIAN')) 
  NOLOGGING 
  TABLESPACE USERS03 
  PCTFREE 10 
  INITRANS 1 
  STORAGE 
  ( 
    BUFFER_POOL DEFAULT 
  ) 
  NOCOMPRESS NO INMEMORY  
);
```
### 创建order_details表的语句是：
```sql
CREATE TABLE ORDER_DETAILS 
(
  ID NUMBER(10, 0) NOT NULL 
, ORDER_ID NUMBER(10, 0) NOT NULL 
, PRODUCT_ID VARCHAR2(40 BYTE) NOT NULL 
, PRODUCT_NUM NUMBER(8, 2) NOT NULL 
, PRODUCT_PRICE NUMBER(8, 2) NOT NULL 
, CONSTRAINT ORDER_DETAILS_FK1 FOREIGN KEY
  (
  ORDER_ID 
  )
  REFERENCES ORDERS
  (
  ORDER_ID 
  )
  ENABLE 
) 
PCTFREE 10 
PCTUSED 40 
INITRANS 1 
STORAGE 
( 
  BUFFER_POOL DEFAULT 
) 
NOCOMPRESS 
NOPARALLEL 
PARTITION BY REFERENCE (ORDER_DETAILS_FK1) 
(
  PARTITION PARTITION_BEFORE_2016 
  LOGGING 
  TABLESPACE USERS 
  PCTFREE 10 
  INITRANS 1 
  STORAGE 
  ( 
    BUFFER_POOL DEFAULT 
  ) 
  NOCOMPRESS NO INMEMORY  
, PARTITION PARTITION_BEFORE_2017 
  LOGGING 
  TABLESPACE USERS02 
  PCTFREE 10 
  INITRANS 1 
  STORAGE 
  ( 
    BUFFER_POOL DEFAULT 
  ) 
  NOCOMPRESS NO INMEMORY  
, PARTITION PARTITION_BEFORE_2018 
  LOGGING 
  TABLESPACE USERS03 
  PCTFREE 10 
  INITRANS 1 
  STORAGE 
  ( 
    BUFFER_POOL DEFAULT 
  ) 
  NOCOMPRESS NO INMEMORY  
);
```
### 表创建成功
![](https://github.com/03DuLi/oracle/blob/master/test3/11.png)
![](https://github.com/03DuLi/oracle/blob/master/test3/22.png)
### 查看数据库的使用情况：
```sql
$ sqlplus system/123@pdborcl
SQL>SELECT tablespace_name,FILE_NAME,BYTES/1024/1024 MB,MAXBYTES/1024/1024 MAX_MB,autoextensible FROM dba_data_files  WHERE  tablespace_name='USERS';

SQL>SELECT a.tablespace_name "表空间名",Total/1024/1024 "大小MB",
 free/1024/1024 "剩余MB",( total - free )/1024/1024 "使用MB",
 Round(( total - free )/ total,4)* 100 "使用率%"
 from (SELECT tablespace_name,Sum(bytes)free
        FROM   dba_free_space group  BY tablespace_name)a,
       (SELECT tablespace_name,Sum(bytes)total FROM dba_data_files
        group  BY tablespace_name)b
 where  a.tablespace_name = b.tablespace_name;
```
### 实验结果：
![](https://github.com/03DuLi/oracle/blob/master/test3/33.png)
![](https://github.com/03DuLi/oracle/blob/master/test3/44.png)
