命名的 PL/SQL 块，编译并存储在数据库中。 

### 子程序
子程序的各个部分：  
>
    声明部分
    可执行部分
    异常处理部分(可选)
子程序的分类：  
>
    过程 － 执行某些操作
    函数 － 执行操作并返回值

子程序的优点：
>
    模块化:将程序分解为逻辑模块;
    可重用性:可以被任意数目的程序调用;
    可维护性:简化维护操作;
    安全性:通过设置权限，使数据更安全.

### 过程
创建过程的语法（和C语言很相似。or replace可以省略。）：
```
CREATE [OR REPLACE] PROCEDURE 
   <procedure name> [(<parameter list>)]
IS|AS 
   <local variable declaration>
BEGIN
   <executable statements>
[EXCEPTION
   <exception handlers>]
END;
```

过程参数的三种模式：
>
    IN：用于接受调用程序的值，默认的参数模式；
    OUT：用于向调用程序返回值； 
    IN OUT：用于接受调用程序的值，并向调用程序返回更新的值。

执行过程的语法：
  `EXECUTE procedure_name(parameters_list);`
  

```
-- IN示范
create or replace procedure proc1(i in number)
as
  a varchar2(50);
begin
  a := '';
  for j in 1..i loop
    a := a || '*';
    dbms_output.put_line(a);
  end loop;
end;
/

exec proc1(6);

---------------------------------------------------------------------
-- OUT示范
create or replace procedure proc2(j out int)
as
begin
  j := 100;
  dbms_output.put_line(j);
end;

只能在另外一个pl/sql段调用:
declare
  k number;
begin
  proc2(k);
end;

在sql*plus运行存储过程，要加‘exec’，而在pl/sql中可以直接调用。

---------------------------------------------------------------------
-- IN OUT示范
create or replace procedure proc3(p1 in out number, p2 in out number)
as
  v_temp number;
begin
  v_temp := p1;
  p1 := p2;
  p2 := v_temp;
end;

declare
  num1 number := 100;
  num2 number := 200;
begin
  proc3(num1,num2);
  dbms_output.put_line(num1);
  dbms_output.put_line(num2);
end;

---------------------------------------------------------------------
show user
exec scott.proc1(6)没有权限
赋权限
grant execute on proc1 to zsr
```

将过程的执行权限授予其他用户：

```
GRANT EXECUTE ON find_emp TO MARTIN;
GRANT EXECUTE ON swap TO PUBLIC;
```
删除过程：

```
DROP PROCEDURE find_emp;
```

### 函数
函数是可以返回值的命名的 PL/SQL 子程序。   
创建函数的语法：
```
CREATE [OR REPLACE] FUNCTION 
  <function name> [(param1,param2)]
RETURN <datatype>  IS|AS 
  [local declarations]
BEGIN
  Executable Statements;
  RETURN result;
EXCEPTION
  Exception handlers;
END;
```

定义函数的限制：
>
    函数只能接受 IN 参数，而不能接受 IN OUT 或 OUT 参数;
    形参不能是 PL/SQL 类型，只能是数据库类型;
    函数的返回类型也必须是数据库类型.
访问函数的两种方式：
>
    使用 PL/SQL 块;
    使用 SQL 语句.
    
```
-- 创建函数
create or replace function fun_hell return varchar2
is
begin
  return '你好，朋友';
end;

调用函数；
sql*plus 
select fun_hello from dual;

pl/sql
declare
  ss varchar2(20);
begin
  ss := fun_hello;
  dbms_output.put_line(ss);
end;
```

#### 函数练习
```
create table 分数表 (student_no number(3),name varchar2(10), score number(3));

insert into  分数表  values (1,'张一', 56);
insert into  分数表 values(2,'张二', 82);
insert into  分数表 values  (3,'张三', 90);
```
要求：创建一个函数，可以接受用户输入的学号，得到该学生的名次，并输出这个名次。


```
create or replace function fun1(sno1 int) return int
is
  score1 number;
  mingci1 number;
begin
  select score into score1 from 分数表 where student_no = sno1;
  select count(*) into mingci1 from 分数表 where score > score1;
  mingci1 := mingci1 + 1;
  return mingci1;
end;
/

--编译错误，查看错误
show error

-- 修改
ed

declare
 mc number;
begin
 mc := func1(2);
 dbms_output.put_line('第' || mc || '名');
end;
```

#### 过程和函数的比较

过程 | 函数
---|---
作为 PL/SQL 语句执行|作为表达式的一部分调用
在规格说明中不包含  RETURN 子句	|必须在规格说明中包含 RETURN 子句
不返回任何值	|必须返回单个值
可以包含 RETURN 语句，但是与函数不同，它不能用于返回值	|必须包含至少一条 RETURN语句

### 自主事务处理
自主事务处理
>
    主事务处理启动独立事务处理;
    然后主事务处理被暂停;
    自主事务处理子程序内的 SQL 操作;
    然后终止自主事务处理;
    恢复主事务处理.
`PRAGMA AUTONOMOUS_TRANSACTION` 用于标记子程序为自主事务处理。

自主事务处理的特征：
>
    与主事务处理的状态无关;
    提交或回滚操作不影响主事务处理;
    自主事务处理的结果对其他事务是可见的;
    能够启动其他自主事务处理.

```
-- 自主事务处理
create or replace procedure p2
as
  a varchar2(50);
  -- 自主事务处理，p1的个更改不影响p2
  PRAGMA AUTONOMOUS_TRANSACTION;
begin
  select sname into a from scott.student where sno=2;
  dbms_output.put_line(a);
end;

create or replace procedure p1
as
  n varchar2(50);
begin
  update scott.student set sname = 'ZSR' where sno=2;
  p2();
  select sname into b from scott.student where sno=2;
  dbms_output.put_line(b);
end;
```