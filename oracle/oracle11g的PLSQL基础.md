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

#### ①数字数据类型
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

#### ②字符数据类型  
CHAR  
VARCHAR2  
LONG  
RAW  
LONG RAW

#### ③日期时间类型  
存储日期和时间数据  
常用的两种日期时间类型
>
    DATE
    TIMESTAMP
#### ④布尔数据类型
>
    此类别只有一种类型，即BOOLEAN类型
    用于存储逻辑值(TRUE、FALSE和NULL)
    不能向数据库中插入BOOLEAN数据  
    不能将列值保存到BOOLEAN变量中  
    只能对BOOLEAN变量执行逻辑操作

### LOB数据类型
用于存储大文本、图像、视频剪辑和声音剪辑等非结构化数据。  
LOB数据类型可存储最大4GB的数据。  
LOB类型包括：  
>
    BLOB        将大型二进制对象存储在数据库中
    CLOB        将大型字符数据存储在数据库中
    NCLOB       存储大型UNICODE字符数据
    BFILE       将大型二进制对象存储在操作系统文件中

LOB类型的数据库列仅存储定位符，该定位符指向大型对象的存储位置


```
create table testClob
(
tid number primary key,
t_info clob
);

insert into testClob values(1,'杜月笙光绪十四年（1888）七月十五，正好中元节那天，出生在上海浦东一户平民家庭。他所家所在的高桥镇，两三千人里，竟然只有谢周两户还算富裕。据章君榖《杜月笙传》记载，所谓的富裕，其实也就是靠着做泥水木匠，衣食无忧而已。杜月笙的父亲杜文卿，跟人在上海杨树浦开了一间米店。叫起来是杜老板，实际上，这生意做得可真是苦。');

select *from testClob;
-- 直接查询，clob类型无法全部显示出来，需要借助PL/SQL
SET SERVEROUTPUT ON
DECLARE
  clob_var   CLOB;
  amount     INTEGER;
  offset     INTEGER;
  output_var VARCHAR2(1000);
BEGIN
  SELECT t_info INTO clob_var FROM testClob WHERE tid=1;
  amount := 1000;  -- 要读取的字符数
  offset := 1;   -- 起始位置
  DBMS_LOB.READ(clob_var,amount,offset,output_var);
  DBMS_OUTPUT.PUT_LINE(output_var);
END;
/
```

```
-- 插入图片与重定向
create or replace procedure insertBlob(id varchar2,imgFile varchar2)
is
  img_file bfile;
  img_blob blob;
  lob_length number;
begin
  -- 先插入一个空值
  insert into person values(id,empty_blob());
  select photo into img_blob from person where pid = id;
  -- 读取img_file中的内容
  img_file := bfilename('PHOTO',imgFile);
  dbms_lob.open(img_file);
  lob_length := dbms_blob.getlengtth(img_file);
  dbms_lob.loadfromfile(img_blob,img_file,lob_length);
  dbms_lob.close(img_file);
  commit;
end;
/

-- 执行存储过程
exec insertBlob('1','test.png');

desc 表名;

-- 构建序列
create sequence seq1 start with 1 increment by 1;

-- 清除屏幕
clear screen;
/

-- 执行存储过程
exec insertBlob('1','test.png');

-- &符号的意义
&grade代表从键盘输入数据
```

### 属性类型
用于引用数据库列的数据类型，以及表示表中一行的记录类型,属性类型有两种：
>
    %TYPE  -  引用变量和数据库列的数据类型
    %ROWTYPE  -  提供表示表中一行的记录类型
使用属性类型的优点：
> 
    不需知道被引用的表列具体类型
    如被引用对象的数据类型发生改变，PL/SQL 变量的数据类型也随之改变，健壮性

### 逻辑比较
逻辑比较用于比较变量和常量的值，这些表达式称为布尔表达式  
布尔表达式由关系运算符与变量或常量组成  
布尔表达式的结果为TRUE、FALSE或NULL，通常由逻辑运算符AND、OR和NOT连接  
布尔表达式有三种类型：
>
    数字布尔型
    字符布尔型
    日期布尔型
    
```
declare 
  sbirth1 student2.sbirthday%type;
begin
  select sbirthday into sbirth1 from student2 where sno =1;
  if sbirth1 < to_date('19900101','yyyymmdd') then
     dbms_output.put_line('大于25岁');
  end if;
  if sbirth1 < to_date('19900101','yyyymmdd') then
     dbms_output.put_line('大于25岁');
  end if;
end;
```
### 控制结构
PL/SQL 支持的流程控制结构：
条件控制
>
    IF 语句
    CASE 语句
循环控制
>
    LOOP 循环
    WHILE 循环
    FOR 循环
顺序控制
>
    GOTO 语句
    NULL 语句

```
declare 
  outgrade varchar2(20);
begin
  outgrade := case &grade
    when 'A' then '优秀'
    when 'B' then '良好'
    when 'C' then '中等'
    when 'D' then '及格'
    when 'E' then '不及格'
    else 	  '没有此成绩'
  end;
  dbms_output.put_line(outgrade);
end;


declare
  j number:=0;
begin
  loop
      j:=j+1;
      dbms_output.put_line(to_char(j)||'---');
      exit      when j>7;
      continue  when j>4;
  end loop;
end;

declare
  j number:=1;
begin
  while j<=8
  loop
      dbms_output.put_line(to_char(j)||'---');
      j := j+1;
  end loop;
end;

declare
  j number:=1;
begin
  for j in 1..8 loop
      dbms_output.put_line(j||'---');
  end loop;
end;

declare
  j number:=1;
begin
  <<aa>>
      dbms_output.put_line(j||'---');
      j := j+1;
      if j <= 7 then goto aa; end if;
      if j > 7 then goto bb; end if;
  <<bb>> null;
end;
```
#### ①条件控制
>
    IF语句根据条件执行一系列语句，有三种形式：IF-THEN、IF-THEN-ELSE 和 IF-THEN-ELSIF
    CASE 语句用于根据单个变量或表达式与多个值进行比较
    执行CASE 语句前，先计算选择器的值

#### ②循环控制
循环控制用于重复执行一系列语句,循环控制语句包括：
>
    LOOP、EXIT 和 EXIT WHEN 、FOR 、WHILE
循环控制的三种类型：
>
    LOOP   -   无条件循环
    WHILE  -  根据条件循环
    FOR  -  循环固定的次数

```
LOOP 
  sequence_of_statements
END LOOP;

WHILE condition LOOP 
  sequence_of_statements
END LOOP;

FOR counter IN [REVERSE] value1..value2
LOOP 
  sequence_of_statements
END LOOP;
```

#### ③顺序控制
顺序控制用于按顺序执行语句,
顺序控制语句包括：
>
    GOTO 语句 -  无条件地转到标签指定的语句
    NULL 语句 -  什么也不做的空语句

#### ④动态SQL
动态 SQL 是指在PL/SQL程序执行时生成的 SQL 语句  
编译程序对动态 SQL 不做处理，而是在程序运行时动态构造语句、对语句进行语法分析并执行  
DDL 语句命令和会话控制语句不能在 PL/SQL 中直接使用，但是可以通过动态 SQL 来执行  
执行动态 SQL 的语法：
>
       EXECUTE IMMEDIATE dynamic_sql_string
          [INTO  define_variable_list]
          [USING bind_argument_list];

含有ddl的时候，要使用动态sql。  
eg.
>
    begin 
      execute immediate 'create table T(t1 int)'
    end;

### 错误处理
在运行程序时出现的错误叫做异常  
发生异常后，语句将停止执行，控制权转移到 PL/SQL 块的异常处理部分  
异常有两种类型：
>
    预定义异常 -  当 PL/SQL 程序违反 Oracle 规则或超越系统限制时隐式引发
    用户定义异常  -  用户可以在 PL/SQL 块的声明部分定义异常，自定义的异常通过 RAISE 语句显式引发
RAISE_APPLICATION_ERROR 过程
>用于创建用户定义的错误信息  
可以在可执行部分和异常处理部分使用  
错误编号必须介于 –20000 和 –20999 之间  
错误消息的长度可长达 2048 个字节  
>- 引发应用程序错误的语法：  
   RAISE_APPLICATION_ERROR(error_number, error_message);
   
```
declare 
  sname1 student.sname%type;
begin
  select sname into sname1 from student where sno =5;
  dbms_output.put_line(sname1);
exception
  when no_data_found then
  dbms_output.put_line('没有这个学生存在');
end;

declare 
  sbirth1 student2.sbirthday%type;
  excep1  exception;
begin
  select sbirthday into sbirth1 from student2 where sno =4;
  if sbirth1 > to_date('20100101','yyyymmdd') then
     raise excep1;
  else
     dbms_output.put_line('学生年龄正常');
  end if;
exception
  when excep1 then
     dbms_output.put_line('4号学生的年龄错误');
end;
```







