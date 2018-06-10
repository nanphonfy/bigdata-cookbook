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

- 算术操作符
>用于执行数值计算  
可以在SQL语句中使用算术表达式，算术表达式由数值数据类型的列名、数值常量和连接它们的算术操作符组成  
包括加(+)、减(-)、乘(*)、除(/)

- 比较操作符
>用于比较两个表达式的值  
包括 =、!=、<、>、<=、>=、BETWEEN…AND、IN、LIKE 和 IS NULL等，LIKE可以使用匹配符_、%

- 逻辑操作符  
>用于组合多个计较运算的结果以生成一个或真或假的结果  
包括与(AND)、或(OR)和非(NOT)  

-连接操作符 
>用于将多个字符串或数据值合并成一个字符串
```
- 通过使用连接操作符可以将表中的多个列合并成逻辑上的一行列
SQL> SELECT (venname|| ' 的地址是 '||venadd1||' '||venadd2 ||' '||venadd3) address
FROM vendor_master WHERE vencode='V001';
```
#### 操作符的优先级
- SQL 操作符的优先级从高到低的顺序是：
>
    算术操作符           --------最高优先级
    连接操作符
    比较操作符
    NOT 逻辑操作符
    AND 逻辑操作符
    OR   逻辑操作符   --------最低优先级 

### Oracle 函数
>Oracle 提供一系列用于执行特定操作的函数  
SQL 函数带有一个或多个参数并返回一个值  

- SQL函数的分类  
`SQL函数->单行函数、分组函数、分析函数`

##### 单行函数分类
>单行函数对于从表中查询的每一行只返回一个值  
可以出现在 SELECT 子句中和 WHERE 子句中   
>- 单行函数可以大致划分为：
>>字符函数  
日期时间函数  
数字函数  
转换函数  
混合函数

- 字符函数

函数 | 输入 |输出
---|---|---
Initcap(char) 	|Select initcap('hello') from dual;	|Hello 
Lower(char) 	|Select lower('FUN') from dual;	|fun 
Upper(char) 	|Select upper('sun') from dual;	|SUN 
Ltrim(char,set) 	|Select ltrim( 'xyzadams','xyz') from dual; 	|adams
Rtrim(char,set) 	|Select rtrim('xyzadams','ams') from dual; |xyzad 
Translate(char, from, to) 	|Select translate('jack','j' ,'b') from dual; 	|back 
Replace(char, searchstring,[rep string]) 	|Select replace('jack and jue' ,'j','bl') from dual;	|black and blue 
Instr (char, m, n) 	|Select instr ('worldwide','d') from dual; 	|5 
Substr (char, m, n) 	|Select substr('abcdefg',3,2) from dual; 	|cd 
Concat (expr1, expr2) 	|Select concat ('Hello',' world') from dual; 	|Hello world

- 字符函数
>以下是一些其它的字符函数：  
CHR和ASCII  
LPAD和RPAD  
TRIM  
LENGTH  
DECODE
```
-- 字符函数接受字符输入并返回字符或数值
select chr(96) from dual;

CHR(96)
-------
`
select ascii('a') from dual;

ASCII('A')
----------
        97

select lpad('abcd',10,'x') from dual;

LPAD('ABCD',10,'X')
-------------------
xxxxxxabcd
```

- 日期时间函数
>日期函数对日期值进行运算，并生成日期数据类型或数值类型的结果  
>- 日期函数包括： 
>>ADD_MONTHS  
MONTHS_BETWEEN  
LAST_DAY  
ROUND  
NEXT_DAY  
TRUNC  
EXTRACT

```
-- oracle取年份，比较麻烦
select extract(year from sysdate) from dual;

-- 这个月的最后一天
select last_day(sysdate) from dual;

-- 大于27岁的员工
select *from employee where add_months(birthday,47*12)<sysdate;

-- 从出生到现在有多少天
select floor(sysdate - birthday) from employee

-- 员工的出生日期为当月最后20天那日
select *from employee where last_day(birthday)-20 = birthday;
```
- 数字函数
数字函数接受数字输入并返回数值结果

函数 | 输入 |输出
---|---|---
Abs(n) |	Select abs(-15) from dual; |	15
Ceil(n) |	Select ceil(44.778) from dual; |	45
Cos(n) 	|Select cos(180) from dual; |	-.5984601 
Cosh(n) |	Select cosh(0) from dual; |	1
Floor(n) |	Select floor(100.2) from dual; |	100
Power(m,n) |	Select power(4,2) from dual; |	16 
Mod(m,n) |	Select mod(10,3) from dual; |	1
Round(m,n)| 	Select round(100.256,2) from dual; |	100.26 
Trunc(m,n)| 	Select trunc(100.256,2) from dual; |	100.25 
Sqrt(n) |	Select sqrt(4) from dual; |	2 
Sign(n)|	Select sign(-30) from dual;|	-1

- 转换函数
>转换函数将值从一种数据类型转换为另一种数据类型
>- 常用的转换函数有：
>>TO_CHAR  
TO_DATE  
TO_NUMBER 

- 混合函数
>DECODE()函数  
>以下是几个用来转换空值的函数：    
>>NVL，第一为空返回二；否则返回一;  
NVL2，第一个不空则返回二；否则返回三。  
NULLIF，两个表达式，相等则返回空；否则第一个  


```
select nvl(2,1) from dual;

  NVL(2,1)
----------
         2
select nvl('',1) from dual;

NVL('',1)
---------
1
select nvl(null,1) from dual;

NVL(NULL,1)
-----------
          1
select nvl2(11,22,33) from dual;

NVL2(11,22,33)
--------------
            22
select nvl2(NULL,22,33) from dual;

NVL2(NULL,22,33)
----------------
              33
select nullif(2,2) from dual;

NULLIF(2,2)
-----------
select nullif(2,23) from dual;

NULLIF(2,23)
------------
           2
```
- 分组函数
>分组函数基于一组行来返回结果  
为每一组行返回一个值


`分组函数->AVG MIN MAX SUM COUNT`

#### GROUP BY和HAVING子句
- GROUP BY子句
>用于将信息划分为更小的组  
每一组行返回针对该组的单个结果

- HAVING子句
>用于指定 GROUP BY 子句检索行的条件

- 练习
```
-- 各部门的总工资
SQL> select department,sum(salary)
from employee
group by department;

DEPARTMENT SUM(SALARY)
---------- -----------
财务科          5800.8
办公室            4000
业务科           18500

-- 各地区的平均工资
select address,avg(salary)
from employee
group by address;

ADDRESS         AVG(SALARY)
--------------- -----------
上海市          2533.333333
南昌市                 3100
南京市               4150.4

-- 平均工资大于3000的部门
select department,avg(salary)
from employee
group by department
having(avg(salary) > 3000);

DEPARTMENT AVG(SALARY)
---------- -----------
财务科          5800.8
办公室            4000

-- 平均工资 大于所有员工平均工资 的部门和平均工资
select department,avg(salary)
from employee
group by department
having(avg(salary) > (select avg(salary) from employee));

DEPARTMENT AVG(SALARY)
---------- -----------
财务科          5800.8
办公室            4000

-- 平均成绩大于 3000 的部门和部门平均工资，按部门名称降序
select department,avg(salary)
from employee
group by department
having(avg(salary) > 3000)
order by department;

DEPARTMENT AVG(SALARY)
---------- -----------
办公室            4000
财务科          5800.8
```

### Oracle 的多表查询
>等值连接  
外连接  
自连接  
子查询

- 相等联接的写法
```SQL  
-- 相等连接(第一种写法)：
select table1.column,table2.column
from  table1, table2 
where  table1.column1=table2.column2

-- 相等连接(第二种写法)：
select table1.column,table2.column
from  table1 inner join table2 
on   table1.column1=table2.column2
```
- 左外联接的写法
```SQL  
-- 左外连接(第一种写法)：
select table1.column,table2.column
from  table1 left  outer  join table2
on   table1.column1=table2.column2

-- 左外连接(第二种写法)：
select table1.column,table2.column
from  table1, table2 
where  table1.column1=table2.column2(+)
```
- 集合操作符  
集合操作符将两个查询的结果组合成一个结果
`集合操作符->UNION UNION ALL INTERSECT MINUS`
>INTERSECT 操作符只返回两个查询的公共行。  
MINUS 操作符返回从第一个查询结果中排除第二个查询中出现的行  

- 重命名
>重命名表：rename table_name1 to table_name2;  
重命名列：alter table table_name rename column col_oldname to colnewname ;
```
--from 产业基地
create table employee(
employee varchar2(10) not null,
employeeName varchar2(10) not null,
sex varchar2(1) not null,
birthday date not null,
address varchar2(15),
telephone number(20) not null,
hiredate date not null,
department varchar2(10),
headship varchar2(10),
salary number(10,2) not null
);
alter table employee add constraint pk_employee primary key(employee);


insert into employee values('E2005001','喻自强','M',to_date('1965-4-15','yyyy-mm-dd'),'南京市','13817605008',to_date('1990-2-6','yyyy-mm-dd'),'财务科','科长','5800.80');
insert into employee values('E2005002','张小梅','F',to_date('1973-11-1','yyyy-mm-dd') ,'上海市','13817605008',to_date('1991-3-28','yyyy-mm-dd'),'业务科','职员','2400.00');
insert into employee values('E2005003','张小娟','F',to_date('1973-3-6','yyyy-mm-dd'),'上海市','13817605008',to_date('1992-3-28','yyyy-mm-dd'),'业务科','职员','2600.00');
insert into employee values('E2005005','张小东','M',to_date('1973-9-3','yyyy-mm-dd'),'南昌市','13817605008',to_date('1992-3-28','yyyy-mm-dd'),'业务科','职员','1800.00');
insert into employee values('E2006001','陈辉','M',to_date('1965-11-1','yyyy-mm-dd'),'南昌市','13817605008',to_date('1990-3-28','yyyy-mm-dd'),'办公室','主任','4000.00');
insert into employee values('E2006002','韩梅','F',to_date('1973-12-11','yyyy-mm-dd'),'上海市','13817605008',to_date('1990-11-28','yyyy-mm-dd'),'业务科','职员','2600.00');
insert into employee values('E2006003','刘风','F',to_date('1973-5-21','yyyy-mm-dd'),'南昌市','13817605008',to_date('1991-2-28','yyyy-mm-dd'),'业务科','职员','2500.00');
insert into employee values('E2007001','吴浮萍','M',to_date('1973-9-12','yyyy-mm-dd'),'南京市','13817605008',to_date('1990-6-28','yyyy-mm-dd'),'业务科','职员','2500.00');
insert into employee values('E2005004','张露','F',to_date('1967-1-5','yyyy-mm-dd'),'南昌市','13817605008',to_date('1990-3-28','yyyy-mm-dd'),'业务科','科长','4100.00');
commit;
```

