### 程序包
>①程序包是对相关过程、函数、变量、游标和异常等对象的封装；  
②程序包由规范和主体两部分组成。

- 程序包：规范+主体  
**规范：** 声明程序包中公共对象。包括类型、变量、常量、异常、游标规范和子程序规范等；  
**主体：** 声明程序包私有对象和实现在包规范中声明的子程序和游标。

>程序包：包头+包体  
包头：公用类型的变量、常量、异常、过程、函数等的声明。  
包体：私有类型的变量、常量、异常、过程、函数等的声明，以及包头内的过程、函数的实现。

```sql  
--程序包规范
CREATE [OR REPLACE]
  PACKAGE 
  package_name IS|AS  
[Public item declarations]  
[Subprogram specification]  
END [package_name];

--程序包主体
CREATE [OR REPLACE] PACKAGE BODY package_name IS|AS  
[Private item declarations]  
[Subprogram bodies]  
[BEGIN
Initialization]  
END [package_name];
```

- 练习
```
--包头
create or replace package pack1
is
  aa int := 9;
  procedure insert_student(a1 in student%rowtype);
  procedure update_student(a2 in student%rowtype);
end pack1;
/

--包体
create or replace package body pack1
is
  bb int := 5;
	procedure insert_student(a1 in student%rowtype)
	is
	begin
	  insert into student(sno,sname,sage) values(a1.sno,a1.sname,a1.sage);
	  commit;
	end insert_student;

	procedure update_student(a2 in student%rowtype)
	is
	begin
	 update student set sname = a2.sname where sno = a2.sno;
	commit;
	end update_student;
end pack1;
/

-- 可以访问包头变量
execute dbms_output.put_line(pack1.aa);
-- 不能访问包体变量
execute dbms_output.put_line(pack1.bb);

-- 创建测试数据
create table student (sno number(3),sname varchar2(10), sage number(3));
insert into student  values (1,'张一', 16);
insert into student values(2,'张二', 15);
insert into student values  (3,'张三', 18);
```

- 使用包头的存储过程
```
declare
  a1 student%rowtype;
begin
  a1.sno := 4;
  a1.sname := '赵飞克';
  a1.sage := 30;
  pack1.insert_student(a1);
end;
/

declare
  a2 student%rowtype;
begin
  a2.sno := 4;
  a2.sname := '赵飞';
  a2.sage := 30;
  pack1.update_student(a2);
end;
/
```
**做项目时，都是：建立程序包，之后再建立过程，函数。**

#### 优点
模块化、更轻松的应用程序设计、信息隐藏、新增功能、性能更佳。

#### 程序包中的游标
>
    ①游标的定义分为游标规范和游标主体两部分；
    ②在包规范中声明游标规范时必须使用 RETURN 子句指定游标的返回类型；
    ③RETURN子句指定的数据类型可以是：
        a.用%ROWTYPE属性引用表定义的记录类型；
        b.程序员定义的记录类型，例如 TYPE EMPRECTYP IS RECORD(emp_id INTEGER,salary REAL)来定义的；
        c.不可以是number, varchar2， %TYPE等类型。 

- 练习
```
--程序包内使用显示游标
create or replace package pack2 is
  cursor mycursor return student%rowtype;
  procedure mycursor_use;
end pack2;
/

create or replace package body pack2 is
  cursor mycursor return student%rowtype is select *from student;
procedure mycursor_use
is 
  stu_rec student%rowtype;
begin
  open mycursor;
  fetch mycursor into stu_rec;
  while mycursor%found loop
    dbms_output.put_line('学号是:' || stu_rec.sno||'，姓名是：'||stu_rec.sname);
    fetch mycursor into stu_rec;
  end loop;
  close mycursor;
end mycursor_use;  
end pack2;
/

exec pack2.mycursor_use;


--程序包内使用ref游标
create or replace package pack3 is
  type refcur is ref cursor;
  procedure mycursor_use;
end pack3;
/

create or replace package body pack3 is
procedure mycursor_use
is 
  mycursor refcur;
  stu_rec student%rowtype;
begin
  open mycursor for select *from student;
  fetch mycursor into stu_rec;
  while mycursor%found loop
    dbms_output.put_line('学号是:' || stu_rec.sno||'，姓名是：'||stu_rec.sname);
    fetch mycursor into stu_rec;
  end loop;
  close mycursor;
end mycursor_use;  
end pack3;
/

exec pack3.mycursor_use;

desc user_source;
```

### 有关子程序和程序包的信息

```
-- USER_OBJECTS 视图包含用户创建的子程序和程序包的信息
SQL> SELECT object_name, object_type
  2  FROM USER_OBJECTS
  3  WHERE object_type IN ('PROCEDURE', 'FUNCTION','PACKAGE', 'PACKAGE BODY');

OBJECT_NAME                                                                      OBJECT_TYPE
-------------------------------------------------------------------------------- -------------------
PACK3                                                                            PACKAGE BODY
PACK3                                                                            PACKAGE
PACK2                                                                            PACKAGE BODY
PACK2                                                                            PACKAGE
PACK1                                                                            PACKAGE BODY
PACK1                                                                            PACKAGE

-- USER_SOURCE 视图存储子程序和程序包的源代码
---注意大写
SQL> SELECT line, text FROM USER_SOURCE
  2  WHERE NAME='PACK3';

      LINE TEXT
---------- --------------------------------------------------------------------------------
         1 package pack3 is
         2   type refcur is ref cursor;
         3   procedure mycursor_use;
         4 end pack3;
         1 package body pack3 is
         2 procedure mycursor_use
         3 is
         4   mycursor refcur;
         5   stu_rec student%rowtype;
         6 begin
         7   open mycursor for select *from student;
         8   fetch mycursor into stu_rec;
         9   while mycursor%found loop
        10     dbms_output.put_line('学号是:' || stu_rec.sno||'，姓名是：'||stu_rec.sname);
        11     fetch mycursor into stu_rec;
        12   end loop;
        13   close mycursor;
        14 end mycursor_use;
        15 end pack3;
```
#### 内置程序包  
>扩展数据库的功能  
为 PL/SQL 提供对 SQL 功能的访问  
用户 SYS 拥有所有程序包  
是公有同义词  
可以由任何用户访问  

### DBMS_Job包的用法
>
    包含以下子过程： 
        Broken()过程：更新一个已提交的工作的状态，典型地是用来把一个已破工作标记为未破工作。这个过程有三个参数：job 、broken与next_date。
    	change()过程：用来改变指定工作的设置。这个过程有四个参数：job、what 、next_date与interval。 
    	Interval()过程：过程用来显式地设置重执行一个工作之间的时间间隔数
    	Isubmit()过程：过程用来用特定的工作号提交一个工作
    	Next_Date()过程：显式地设定一个工作的执行时间。这个过程接收两个参数：job与next_date
        Remove()过程：删除一个已计划运行的工作。这个过程接收一个参数
        Run()过程：用来立即执行一个指定的工作。这个过程只接收一个参数
    	Submit()过程：工作被正常地计划好
    	User_Export()过程：返回一个命令，此命令用来安排一个存在的工作以便此工作能重新提交
    	What()过程：应许在工作执行时重新设置此正在运行的命令

- 练习

```
--创建测试表
create table a(a date);
--创建一个自定义过程
create or replace procedure test as
begin
  insert into a values(sysdate);
end;
/

--每分钟调用test过程一次
--创建JOB  
variable job1 number;    
begin
  --每天1440（24*60）分钟，即一分钟运行test过程一次
  dbms_job.submit(:job1,'test;',sysdate,'sysdate+1/1440');
end;
/
--运行JOB
begin
  dbms_job.run(:job1);
end;
/
--检查结果
select to_char(a,'yyyymmdd hh24:mi:ss') from a;
--删除job
--ORA-01008: 并非所有变量都已绑定(要把":"去掉)
begin
  dbms_job.remove(:job2);
end;
/
--参看jobs
select *from dba_jobs;
select *from dba_jobs_running;
```
### UTL_FILE包的用法
- 练习(未测试)
```
--systemm/systemm
select *from all_directories;
--删除掉directory
drop directory;

create directory TEST_DIR as 'D:\data';
grant read,write on directory TEST_DIR to zsr;

--zsr/zsr
create or replace procedure read_txtfile(path in varchar2,name varchar2)
as
  l_output utl_file.file_type;
  str varchar2(2000);
begin
  l_output := utl_file.fopen(path,name,'r',2000);
  loop
    utl_file.get_line(l_output,str);
	dbms_output.put_line(str);
  end loop;
  utl_file.fclose(l_output);
exception
 when no_data_found then
   utl_file.fclose(l_output); 
end
/

exec read_txtfile('TEST_DIR','1.txt')
```