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
