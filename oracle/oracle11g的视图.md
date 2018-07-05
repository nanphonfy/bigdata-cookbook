### 目标
>学会创建视图，使用视图  
明白键保留表的含义和使用方式

### 视图
>视图以经过定制的方式显示来自一个或多个表的数据  
视图可以视为“虚拟表”或“存储的查询”  
创建视图所依据的表称为“基表”  
视图的优点有：  
>>提供了另外一种级别的表安全性  
隐藏的数据的复杂性  
简化的用户的SQL命令  
隔离基表结构的改变  
通过重命名列，从另一个角度提供数据

- 创建视图语法
```SQL
CREATE [OR REPLACE] [FORCE] VIEW
view_name [(alias[, alias]...)] 
AS select_statement
[WITH CHECK OPTION]
[WITH READ ONLY];
```

### 视图上的dml语句
>在视图上也可以使用修改数据的DML语句，如INSERT、UPDATE和DELETE  
视图上的DML语句有如下限制：  
>>只能修改一个底层的基表  
如果修改违反了基表的约束条件，则无法更新视图  
如果视图包含连接操作符、DISTINCT 关键字、集合操作符、聚合函数或 GROUP BY 子句，则将无法更新视图  
如果视图包含伪列或表达式，则将无法更新视图

#### 视图中的函数
```SQL
--视图中可以使用单行函数、分组函数和表达式
CREATE VIEW item_view AS 
SELECT itemcode, LOWER(itemdesc) item_desc
FROM itemfile; 

--使用DROP VIEW语句删除视图
DROP VIEW toys_view;
```

### 练习
```SQL
--比如移动机房一个表有上百列，用户来查不可能把一百多列都查一遍，
--用户要查询哪几列创建对应的视图。

--物化视图叫做快照，真正存储数据。
--force，表不存在也可以创建
create force view view1 as
select *from employee；

--查看拥有的视图
select *from user_views;

select *from view1;

create view view2 as
select *from employee 
where employee = 'E2005001';

select *from view2;

--视图能否更新
--视图的更新会影响基表
update view2 set employee='T2005001' where employee='E2005001';


create view view3 as
select *from employee
where employee='E2005001'
with check option;

--视图不让更新，减少视图结果集的操作
SQL> update view2 set employee='D2005001' where employee='E2005001';
1 row updated

SQL> update view3 set employee='D2005001' where employee='E2005001';
0 rows updated

--创建不可更新的视图
create view view4 as
select *from employee 
with read only;

select *from view4;
EMPLOYEE   EMPLOYEENAME SEX BIRTHDAY    ADDRESS                     TELEPHONE HIREDATE    DEPARTMENT HEADSHIP         SALARY
---------- ------------ --- ----------- --------------- --------------------- ----------- ---------- ---------- ------------
D2005001   喻自强       M   1965/4/15   南京市                    13817605008 1990/2/6    财务科     科长            5800.80
E2005002   张小梅       F   1973/11/1   上海市                    13817605008 1991/3/28   业务科     职员            3400.00


--可以带order by
create view view5 as
select *from employee
order by hiredate desc;

--【单表视图支持更新】

--多表连接的视图不支持更改。
create view view_student_grade
as
select s.sno s_sno,s.sname,g.km,g.score,g.sno g_sno
from student s,grade g
where s.sno=g.sno;
 
select *from view_student_grade;

S_SNO SNAME      KM              SCORE      G_SNO
----- ---------- ---------- ---------- ----------
    1 张一       语文               65          1
    2 张二       数学               76          2
    3 张三       英语               86          3
    4 赵飞       语文               94          4

update view_student_grade set s_sno=11 where s_sno=1
ORA-01779: 无法修改与非键值保存表对应的列

update view_student_grade set g_sno=11 where g_sno=1
ORA-01779: 无法修改与非键值保存表对应的列

--多表视图就是复杂视图
eg.学生表和部门表
进行连接后的视图，主键是学号，而部门号（部门表的主键）在视图里面就不是主键了。
即学生表是"键保留表"，部门表是"非键保留表"。

--修改 键保留表数据
update ...

--修改 非键保留表数据

--instead of 触发器可以任意更改视图，没有限定 键保留表 和 非键保留表
--可以在创建视图的时候，携带函数
create view view8 as
select sno,upper(sname) new_name from student;

select ... new_name = 'BOB';

--为什么mysql中很少见到使用视图功能？
https://www.zhihu.com/question/26614127
```

