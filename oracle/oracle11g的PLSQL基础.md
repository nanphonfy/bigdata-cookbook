###  简介  
PL/SQL是oracle的核心。
>
    ①PL/SQL 是过程语言(Procedural Language)与结构化查询语言(SQL)结合而成的编程语言。
    （oracle自身提供了很多程序包，也可实现邮件发送、校验等）
    ②PL/SQL 是对 SQL 的扩展；
    ③支持多种数据类型，如大对象和集合类型，可使用条件和循环等控制结构；
    ④可用于创建存储过程、触发器和程序包，给SQL语句的执行添加程序逻辑；
    ⑤与 Oracle 服务器和 Oracle 工具紧密集成，具备可移植性、灵活性和安全性。
    

### 优点
支持 SQL，在 PL/SQL 中可以使用：
>
    数据操纵命令
    事务控制命令
    游标控制
SQL 函数和 SQL 运算符；  
用户把PL/SQL块整个发送到服务器端，oracle服务器端编译、运行，再把结果返回给用户（可节省网络流量）；  
可移植性，可运行在任何操作系统和平台上的Oralce 数据库；  
更佳的性能，PL/SQL 经过编译执行；
安全性，可以通过存储过程限制用户对数据的访问;  
与 SQL 紧密集成，简化数据处理。
>
    支持所有 SQL 数据类型
    支持 NULL 值
    支持 %TYPE 和 %ROWTYPE 属性类型

### 体系结构
>PL/SQL 引擎驻留在 Oracle 服务器中，该引擎接受 PL/SQL 块并对其进行编译执行。

`将PL/SQL 块发送给 Oracle 服务器->Oracle 服务器{[PL/SQL引擎（过程语言执行器）],[SQL引擎（SQL语句执行器）]}->将结果发送给用户`

### PL/SQL 块简介
PL/SQL 块是构成 PL/SQL 程序的基本单元，将逻辑上相关的声明和语句组合在一起。  
PL/SQL 分为三个部分，声明部分、可执行部分和异常处理部分。
```
[DECLARE 
declarations]
BEGIN
executable statements
[EXCEPTION 
handlers]
END;
```

### 常量和变量
PL/SQL 块中可以使用变量和常量
>
    在声明部分声明，使用前必须先声明
    声明时必须指定数据类型，每行声明一个标识符
    在可执行部分的 SQL语句和过程语句中使用
   
声明变量和常量的语法：  
`identifier [CONSTANT] datatype [NOT NULL] [:= | DEFAULT expr];`

给变量赋值有两种方法：
>
    使用赋值语句 :=
    使用 SELECT INTO 语句

给变量赋值的区别：
```
T/SQL
select @a=3
set    @a=3
select @a = count(*) from student;

PL/SQL
a := 3
select count(*) into a from student;
```

### PL/SQL 支持的内置数据类型
PL/SQL 支持的内置数据类型  
>{**标量类型**[数字、字符、布尔型、日期时间]、  
**LOB类型**（存储非结构化数据块）[BFILE、BLOB、CLOB、NCLOB]、  
**属性类型**[%TYPE(提供某个变量或数据库表列的数据类型
)、%ROWTYPE
（提供表示表中一行的记录类型 
）]}

#### 数字数据类型
**BINARY_INTEGER**
（存储有符号整数，所需存储空间少于NUMBER类型值
）->NATURAL
、NATURALLN
、POSITIVE
、POSITIVEN
、SIGNTYPE  
**NUMBER**（存储整数、实数和浮点数
）->DEMICAL、FLOAT、INTEGER、REAL 
**PLS_INTEGER**
（存储有符号整数，可使算术计算快速而有效
）  

#### 字符数据类型  
CHAR  
VARCHAR2  
LONG  
RAW  
LONG RAW

日期时间类型  
存储日期和时间数据  
常用的两种日期时间类型
>
    DATE
    TIMESTAMP
布尔数据类型
>
    此类别只有一种类型，即BOOLEAN类型
    用于存储逻辑值(TRUE、FALSE和NULL)
    不能向数据库中插入BOOLEAN数据  
    不能将列值保存到BOOLEAN变量中  
    只能对BOOLEAN变量执行逻辑操作





