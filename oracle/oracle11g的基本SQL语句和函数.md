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
