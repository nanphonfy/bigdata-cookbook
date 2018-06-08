### 掌握点
>①oracle基本SQL语句；  
②oracle单值、分组函数；  
③oracle多表查询、集合运算。

### 基本SQL语句
>SQL 是 Structured Query Language（结构化查询语言）的首字母缩写词
SQL 是数据库语言，Oracle使用该语言存储和检索信息；表是主要的数据库对象，用于存储数据。

`用户->发送SQL->oracle服务器->发送命令输出到用户端->用户`

#### SQL简介
- SQL 支持下列类别的命令
>
    数据定义语言（DDL）->create alter drop
    数据操纵语言（DML）->insert select delete update
    事务控制语言（TCL）->commit savepoint rollback
    数据控制语言（DCL）->grant revoke

### 数据类型
创建表时，必须为各个列指定数据类型。以下是 Oracle 数据类型的类别：
>
    数据类型：字符、数值、日期时间、RAW/LONG RAW、LOB
    字符数据类型：CHAR、VARCHAR2、LONG
    LOB->CLOB、BLOB、BFILE

- 练习
```SQL  
select *from number_test;
create table number_test (num number(3,2));
--表示所定义的数字最大是3位长，其中包含2位小数。就是说这个类型最大可设置1位整数和2位小数。
insert into number_test values(1.23);
insert into number_test values(1.24767)

SQL> select *from number_test;
  NUM
-----
 1.23
 1.23
 1.24
 
ORA-01438: 值大于为此列指定的允许精度;
insert into number_test values(11.236767);


create table number_test2(num number(3));
insert into number_test2 values(11.23);
insert into number_test2 values(11.67);
-- 会进行四舍五入

SQL> select *from number_test2;

 NUM
----
  11
  12

SQL> select sysdate from dual;
SYSDATE
-----------
2018/6/5 8:

SQL> select to_char(sysdate,'yyyymmdd hh24:mi:ss') from dual;
TO_CHAR(SYSDATE,'YYYYMMDDHH24:
------------------------------
20180605 08:40:22

SQL> select to_char(systimestamp,'yyyymmdd hh24:mi:ssxff6') from dual;

TO_CHAR(SYSTIMESTAMP,'YYYYMMDD
------------------------------
20180605 08:41:05.615000
```

    
>
    Oracle中伪列就像一个表列，但是它并没有存储在表中
    伪列可以从表中查询，但不能插入、更新和删除它们的值
    常用的伪列有ROWID和ROWNUM

>- ROWID 是表中行的存储地址，该地址可以唯一地标识数据库中的一行，可以使用 ROWID 伪列快速地定位表中的一行
>- ROWNUM 是查询返回的结果集中行的序号，可以使用它来限制查询返回的行数

#### 数据定义语言
>数据定义语言用于改变数据库结构，包括创建、更改和删除数据库对象。  
用于操纵表结构的数据定义语言命令有：
>>CREATE TABLE  
ALTER TABLE  
TRUNCATE TABLE  
DROP TABLE

- 练习
```
--创建测试表
create table student0(sno number(6),sname varchar2(10));

--新增列
alter table student0 add tele varchar2(11);

--修改列大小
alter table student0 modify tele varchar2(20);
-- 20 调整回11 也可以 

-- 删除列
alter table student0 drop column tele;

insert into student0 values(1,'A','1389124890');
insert into student0 values(1,'B','1389124890');

--克隆表数据
create table student2 as select *from student0;

--克隆没有数据的表
create table student2 as select *from student where 1>1;

--【删除数据区别】
truncate table student2;
truncate后不需再commit

delete from student2
①需要再commit②会进入日志，可回滚。

-- 把student0表数据插入student2
insert into student2 select *from student0;

-- oracle是区分大小写的
-- 如果不知道是不是大小写，可以这么做
select *from student2 where upper(sname) = 'A';

--插入单引号
update student2 set sname = 'A''C' where sname = 'A';
```
#### 数据操作语言
>数据操纵语言用于检索、插入和修改数据
数据操纵语言是最常见的SQL命令  
数据操纵语言命令包括：  
>>SELECT  
INSERT  
UPDATE  
DELETE

#### 数据控制语言
>数据控制语言为用户提供权限控制命令 
用于权限控制的命令有：
>>GRANT 授予权限  
REVOKE 撤销已授予的权限

### SQL操作符
Oracle 支持的SQL操作符分类如下：算术操作符、比较操作符、逻辑操作符、集合操作符、连接操作符。
