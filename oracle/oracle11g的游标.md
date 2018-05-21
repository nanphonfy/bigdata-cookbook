### 简介
>什么情况下，一定要使用游标，什么情况下，不使用游标也行。能够根据不同情况，选取不同的游标进行使用。

显式游标是用的最广泛的。

>oracle在进行取行记录的时候，实际上内部使用了游标。  
只要进行select操作，内部使用游标，相当于指针。

`oracle服务器->执行PL/SQL程序->检索行->保存到游标中（内存单元）->提取行->一次处理一行`

不使用游标，没有办法单独的一行一行控制。

>
    eg.需求：A表存储了邮件地址，B表是从网上爬的邮件地址（内含一些错误邮箱），要实现从B表剔除不合法的邮箱，当A表不含有该邮箱记录，则插入A表的需求。

逐行处理查询结果，以编程的方式访问数据
。  

游标的类型：
>
    1，隐式游标：在 PL/SQL 程序中执行DML SQL 语句时自动创建隐式游标，名字固定叫sql；
    2，显式游标：显式游标用于处理返回多行的查询；
    3，REF 游标：REF 游标用于处理运行时才能确定的动态 SQL 查询的结果
。

### 隐式游标
>
    ①在PL/SQL中使用DML语句时自动创建隐式游标;
    ②隐式游标自动声明、打开和关闭，其名为 **SQL**;
    ③通过检查隐式游标的属性可以获得最近执行的DML 语句的信息;
    ④隐式游标的属性有：
        %FOUND – SQL 语句影响了一行或多行时为 TRUE；
        %NOTFOUND – SQL 语句没有影响任何行时为TRUE；
        %ROWCOUNT – SQL 语句影响的行数；
        %ISOPEN  - 游标是否打开，始终为FALSE。


```
<!--SQL%FOUND：只有在 DML 语句影响一行或多行时，才返回 True-->
SET SERVEROUTPUT ON
BEGIN
    UPDATE toys SET toyprice=270
    WHERE toyid= 'P005';
    IF SQL%FOUND THEN
    	DBMS_OUTPUT.PUT_LINE(‘表已更新');
    END IF;
END;
/

<!--SQL%NOTFOUND：如果 DML 语句不影响任何行，则返回 True-->
SET SERVEROUTPUT ON
DECLARE
  	v_TOYID TOYS.ID%type := '&TOYID';
  	v_TOYNAME TOYS.NAME%Type := '&TOYNAME';
BEGIN
  	UPDATE TOYS SET NAME = v_TOYNAME
  	WHERE toyid=v_TOYID;
  	IF SQL%NOTFOUND THEN   
  	    DBMS_OUTPUT.PUT_LINE('编号未找到。');
 	ELSE
	    DBMS_OUTPUT.PUT_LINE(‘表已更新');
	END IF;
END;
/
```
#### SELECT INTO 语句
```
<!--NO_DATA_FOUND：如果没有与SELECT INTO语句中的条件匹配的行，将引发NO_DATA_FOUND异常-->
SET SERVEROUTPUT ON
DECLARE 
	empid VARCHAR2(10);
	desig VARCHAR2(10);
BEGIN
	empid:= '&Employeeid';
	SELECT designation INTO desig 
	FROM employee WHERE empno=empid;
    EXCEPTION
	    WHEN NO_DATA_FOUND THEN
	    DBMS_OUTPUT.PUT_LINE('职员未找到');
END;
/

<!--TOO_MANY_ROWS：如果 SELECT INTO 语句返回多个值，将引发TOO_MANY_ROWS异常-->
SET SERVEROUTPUT ON
DECLARE 
	empid VARCHAR2(10);
BEGIN
	SELECT empno INTO empid FROM employee;
    EXCEPTION
	WHEN TOO_MANY_ROWS THEN
	  DBMS_OUTPUT.PUT_LINE('该查询提取多行');
END;
/
```
### 显示游标  
显式游标在 PL/SQL块的声明部分定义查询，该查询可以返回多行。  
显式游标的操作过程:  
`数据库->打开游标->select *from student->提取行->变量->关闭游标`

>声明游标、打开游标、使用游标取出记录、关闭游标。

```
SET SERVER OUTPUT ON
<!--声明游标-->
DECLARE
   my_toy_price  toys.toyprice%TYPE;				  CURSOR toy_cur IS
   SELECT toyprice FROM toys
   WHERE toyprice<250;
BEGIN
   <!--打开游标-->
   OPEN toy_cur;  
     LOOP
        <!--提取行-->
         FETCH toy_cur INTO my_toy_price;
         EXIT WHEN toy_cur%NOTFOUND;
         DBMS_OUTPUT.PUT_LINE 
      ('TOYPRICE=:玩具单价=：'||my_toy_price);
     END LOOP;
     <!--关闭游标-->
     CLOSE toy_cur;
END;

declare
  stu1 student%rowtype;
  cursor mycursor is select * from student;
begin
  open mycursor;
  fetch mycursor into stu1;
  while mycursor%found loop
    dbms_output.put_line('学号是：'||stu1.sno||',姓名是:'||stu1.sname);
    fetch mycursor into stu1;
  end loop;
  close mycursor;
end;
/
```
#### 带参数的显示游标
>
    ①声明显式游标时可以带参数以提高灵活性；
    ②声明带参数的显式游标的语法如下：
    `CURSOR <cursor_name>(<param_name> <param_type>)
         IS select_statement;`
    ③允许使用游标删除或更新活动集中的行；
    ④声明游标时必须使用 SELECT … FOR UPDATE语句。

```
declare
  stu1 student%rowtype;
  sno1 student.sno%type;
  cursor mycursor（input_no number） is select * from student where sno > input_no;
begin
  <!--从键盘输入-->
  sno1 := &学生学号：
  open mycursor;
  fetch mycursor into stu1;
  while mycursor%found loop
    dbms_output.put_line('学号是：'||stu1.sno||',姓名是:'||stu1.sname);
    fetch mycursor into stu1;
  end loop;
  close mycursor;
end;
/
```
#### 使用显示游标更新行
```
CURSOR <cursor_name> IS SELECT statement FOR UPDATE;

UPDATE <table_name>
SET <set_clause>
WHERE CURRENT OF <cursor_name>
	
DELETE FROM <table_name>
WHERE CURRENT OF <cursor_name>
```

```
declare
  stu1 student%rowtype;
  cursor mycursor is select * from student where sno=2 or sno=3 for update;
begin
  open mycursor;
  fetch mycursor into stu1;
  while mycursor%found loop
    update student set sno=sno+100 where current of mycursor;
    fetch mycursor into stu1;
  end loop;
  close mycursor;
end;
```
### 循环游标
>
    ①循环游标用于简化游标处理代码；
    ②当用户需要从游标中提取所有记录时使用；
    ③循环游标的语法如下：
    FOR <record_index> IN <cursor_name>
    LOOP
    	<executable statements>
    END LOOP;

**循环游标只能用于查询，而不能用于更新。**

```
declare
  stu1 student%rowtype;
  cursor mycursor is select * from student;
begin
  for cur_2 in mycursor loop
    dbms_output.put_line('学号是：'||stu1.sno||',姓名是:'||stu1.sname);
  end loop;
end;
/
```
#### fetch ... bulk collect into
适用于大数据量，速度远远快于普通的fetch ... into。


```
declare
  //多列值
  cursor  my_cursor is select ename from emp where deptno=10;
  //声明类型为一张表(只有一个字段)
  type  ename_table_type is table of varchar2(10);
  //相当于一张表
  ename_table  ename_table_type;
begin
  open  my_cursor;
  //批量导入临时表
  fetch my_cursor bulk collect into  ename_table;
  for  i in 1..ename_table.count  loop
     dbms_output.put_line(ename_table(i));
  end  loop;
  close my_cursor;
end;
```
