### 掌握点
>①SQL语句执行顺序，能分析复杂SQL语句的执行过程；  
②Oracle子查询、自连接、行列转换；  
③分析函数、decode函数、 SELECT CASE WHEN的使用；  
④Oracle 分页、删除重复记录的方法。

#### SQL语句的执行顺序
>- 常见的select、from、where的顺序：
>>1, from  2, where  3, select
>- 完整的select、from、where、group by、having、order by的顺序：
>>1, from  2, where  3, group by  4,having  5, select  6, order by

#### EXISTS 的使用
>EXISTS用来判断查询所得的结果中，是否有
满足条件的纪录存在。  
>
    例:select *　from student 
       where exists(select * from address where zz='郑州');
>从select 、from、where三者的先后执行顺
序来分析：
>>判断address表存不存在地址为郑州的列，如果 存在，则返回所有student表数据，否则不返回student表数据（exists返回真或假）。

#### group by的练习

```SQL
create table s(no number,name varchar2(10),age int);
insert into s values (1,'A',21);  
insert into s values (2,'B',22);
insert into s values (3,'A',23);  
insert into s values (4,'A',24);
insert into s values (5,'A',25);  
insert into s values (6,'C',26);
insert into s values (7,'B',27);

-- 查找姓名相同的学生，显示所有信息 
select name,count(1) from s group by name;

NAME         COUNT(1)
---------- ----------
A                   4
B                   2
C                   1

select * from s where s.name in 
(select name from s group by name having(count(*)>1))

        NO NAME                                           AGE
---------- ---------- ---------------------------------------
         1 A                                               21
         2 B                                               22
         3 A                                               23
         4 A                                               24
         5 A                                               25
         7 B                                               27
```
#### 自连接的使用 
```sql
CREATE TABLE  manager  (
  no  char(10) ,
  name  varchar2(10) ,
  manager_no  char(10)
);

insert into manager values('001', '张一', '004');
insert into manager values('002', '张二', '004');
insert into manager values('003', '张三', '003');
insert into manager values('004', '张四', '004');
commit;

-- 显示：编号，姓名，管理人员姓名
select a.no,a.name,b.name from manager a,manager b
where a.manager_no = b.no;

NO         NAME       NAME
---------- ---------- ----------
003        张三       张三
004        张四       张四
002        张二       张四
001        张一       张四
```
#### SELECT CASE WHEN的的使用
- 语法
>CASE   
WHEN   条件1  THEN  action1    
WHEN   条件2   THEN   action2   
WHEN   条件3   THEN   action3   
...  
ELSE actionN   
END CASE

>CASE selector  
WHEN value1 THEN action1  
WHEN value2 THEN action2  
WHEN value3 THEN action3  
...  
ELSE actionN  
END [CASE]

- 练习

```SQL
select  case 
      when  substr('20090310',5,2) = '01'  then  '一月份'
      when  substr('20090310',5,2) = '02'  then  '二月份'
      when  substr('20090310',5,2) = '03'  then  '三月份'
      when  substr('20090310',5,2) = '04'  then  '四月份'
      else null
    end 
from dual;

select  case substr('20090310',5,2) 
      when   '01'  then  '一月份'
      when    '02'  then  '二月份'
      when   '03'  then  '三月份'
      when    '04'  then  '四月份'
      else null
    end 
from dual;

CASEWHENSUBSTR('20090310',5,2)
------------------------------
三月份

-- 把每个学生的grade列，用相应的等级来更新
create table grade(sno number, km varchar2(10), score number,grade char(6));     
insert into  grade values(1, '语文', 65,null);
insert into  grade values(2, '数学', 76,null);
insert into  grade values(3, '英语', 86,null);
insert into  grade values(4, '语文', 94,null);
commit;

update grade a set a.grade = (
select grade from (
	select sno ,
	case  when  score >= 90 then '优秀'
		 when  score >= 80 then '良好'
		 when  score >= 70 then '中等'
		 when  score >= 60 then '及格' 
		 else  '不及格'
	end grade
	from grade
) g
where g.sno = a.sno);
```
#### 复杂更新语句的使用

```SQL
--表T1里有a,b,c,d,e五个字段，表T2里有 a,b,c三个字段，当T1中"c"与表T2中"c"相同，从表T2中将a,b覆盖表T1中的a,b
create table T1(a  int ,b int ,c int ,d int ,e int);
create table T2(a int ,b int ,c int );

insert into T1 values(1,2,3,4,5);
insert into T1 values(10,20,3,4,5);
insert into T1 values(10,20,4, 40,50);
insert into T2 values( -1, -1 , 3);
insert into T2 values( -2, -2, 4);

update t1 set a= (select a from t2 where t2.c = t1.c) ,
b =(select b from t2 where t2.c = t1.c)
where t1.c in (select c from t2);
```
### 分析函数
> ①用于计算完成聚集的累计排名、序号等；
②为每组记录返回多个行。   
>- 以下三个分析函数用于计算一个行在一组有序行中的排位，序号从1开始
>>ROW_NUMBER 返回连续的排序，不论值是否相等  
RANK 具有相等值的行排序相同，序数随后跳跃  
DENSE_RANK 具有相等值的行排序相同，序号是连续的

```SQL
select sno,km,score,
row_number() over(order by score desc)
from grade;

select sno,km,score,
rank() over(order by score desc)
from grade;

select sno,km,score,
dense_rank() over(order by score desc)
from grade;
```
### DECODE 中的if-then-else逻辑
>在逻辑编程中，经常用到If – Then –Else 进行逻辑判断。在DECODE的语法中，实际上就是这样的逻辑处理过程。它的语法如下：    
`DECODE(value, if1, then1,  if2,then2, if3,then3,  . . .  else )`
>>- Value代表某个表的任何类型的任意列或一个通过计算所得的任何结果。
>>- 当每个value值被测试，如果value的值为if1，Decode 函数的结果是then1；
>>- 如果value等于if2，Decode函数结果是then2；
>>- 如果value结果不等于给出的任何配对时，Decode 结果就返回else。

>需要注意的是，这里的if、then及else 都可以是函数或计算表达式。

```SQL
create table s(id number,name varchar2(10),sex char(1));
insert into s values(1, '张', '1');
insert into s values(2,  '王', '2');
insert into s values(3, '李', '1');

select name ,decode(sex, '1','男生', '2','女生') 
from s;
select name ,decode(sex, '1','男生',女生') 
from s;

SQL> select sign(1) from dual;

   SIGN(1)
----------
         1

SQL> select sign(-1) from dual;

  SIGN(-1)
----------
        -1

SQL> select sign(0) from dual;

   SIGN(0)
----------
         0
		 
		 
Create table sales(month char(2),sales_tv number,sales_computer number);
Insert into sales values('01', 10, 18);
Insert into sales values('02', 28, 20);
Insert into sales values('03', 36, 33);
 
-- DECODE取出一行内两列中的较大值
select month, 
decode(sign(sales_tv -sales_computer), 1, sales_tv, sales_computer) as 较大销售量 
from sales; 
```
#### Oracle中的行列转换
```SQL
格式1：
商品名称   季度        销售额
---------- ---- ----------
电视机     01          100
电视机     02          200
电视机     03          300
空调       01           50
空调       02          150
空调       03          180

格式2：
商品名称          一季度        二季度        三季度        四季度
---------- ---------- ---------- ---------- ----------
电视机            100        200        300          0
空调               50        150        180          0
```
```SQL
create table sales(name varchar2(10), quarter char(2), sales number);
insert into sales values('电视机', '01', 100);
insert into sales values('电视机', '02', 200);
insert into sales values('电视机', '03', 300);
insert into sales values('空调', '01', 50);
insert into sales values('空调', '02', 150);
insert into sales values('空调', '03', 180);

--从格式1到格式2：
select a.name,
     sum(decode(a.quarter,'01', a.sales ,0  ))  一季度,
     sum(decode(a.quarter,'02', a.sales ,0  ))  二季度,
     sum(decode(a.quarter,'03', a.sales ,0  ))  三季度,
     sum(decode(a.quarter,'04', a.sales ,0  ))  四季度
from sales a 
group by a.name 
order by 1;
```
#### ROWNUM 的使用
- between and 只能从1开始，否则取不到记录

[为什么oracle中rownum只能小于，不能大于](https://zhidao.baidu.com/question/590128041.html)

```
-- 记得，rownum不能取大于1的，所以要取别名
select bh,gz 
from (
  select yggz.*, rownum rn
  from yggz 
)
where rn >=3 and rn <= 5;
```
- 查找表中，第3条到第5条记录，并显示出来。
```SQL
-- 第一种写法
select * from yggz where rownum<=5
minus
select * from yggz where rownum<=2;

     BH         GZ
------- ----------
      3        900
      4       2000
      5       1500
	  
select yggz.*, rownum rn
from yggz;

     BH         GZ         RN
------- ---------- ----------
      1       1000          1
      2       1100          2
      3        900          3
      4       2000          4
      5       1500          5
      6       3000          6
      7       1400          7
      8       1200          8
      
-- 第二种写法
select bh,gz 
from (
  select yggz.*, rownum rn
  from yggz 
)
where rn >=3 and rn <= 5;

--创建yggz表（员工工资表）
create table yggz (
    bh number(6),
    gz number
);

insert into yggz values(1,1000);
insert into yggz values(2,1100);
insert into yggz values(3,900);
insert into yggz values(4,2000);
insert into yggz values(5,1500);
insert into yggz values(6,3000);
insert into yggz values(7,1400);
insert into yggz values(8,1200);
```
- 按工资由高到底，查找表中，第3高的到第5高的记录，并显示出来
```SQL
select *from yggz order by gz desc;

     BH         GZ
------- ----------
      6       3000
      4       2000
      5       1500
      7       1400
      8       1200
      2       1100
      1       1000
      3        900
      
-- 记得要对rownum取别名
select bh,gz from (
    select a.*,rownum rn from(
    select yggz.* from yggz 
    order by gz desc  ) a 
)
where rn<=5 and rn>2;

-- 或第二种写法
select * from (
    select * from yggz
    order by gz desc)       
where rownum<=5
minus
select * from (
    select * from yggz
    order by gz desc)      
where rownum<=2;
```
### 删除重复记录

- rowid里的信息有 数据库对象号、数据文件号、数据块号、行号。rowid查询是最快的，比索引还快。
```
select student.*,rowid from student;
 SNO SNAME      SAGE TELE        ROWID
---- ---------- ---- ----------- ------------------
   1 张一         16             AAAE5mAABAAAK/hAAA
   2 张二         15             AAAE5mAABAAAK/hAAB
   3 张三         18             AAAE5mAABAAAK/hAAC
   4 赵飞         30             AAAE5mAABAAAK/hAAD
   5 润之         20             AAAE5mAABAAAK/hAAE
   6 陈独秀       34             AAAE5mAABAAAK/hAAF
   7 李大釗       35             AAAE5mAABAAAK/hAAG
   8 革命         35             AAAE5mAABAAAK/hAAH
```

```SQL
create table s(sno number(6)  , sname varchar2(10), sage  int );  
insert into s values(1, 'AA', 21);
insert into s values(2, 'BB', 22);
insert into s values(3, 'CC', 23);
insert into s values(3, 'CC', 34);
insert into s values(3, 'CC', 35);
insert into s values(3, 'CC', 36);

select s.*,rowid from s;

    SNO SNAME                                         SAGE ROWID
------- ---------- --------------------------------------- ------------------
      1 AA                                              21 AAAE9JAABAAALBxAAA
      2 BB                                              22 AAAE9JAABAAALBxAAB
      3 CC                                              23 AAAE9JAABAAALBxAAC
      3 CC                                              34 AAAE9JAABAAALBxAAD
      3 CC                                              35 AAAE9JAABAAALBxAAE
      3 CC                                              36 AAAE9JAABAAALBxAAF

-- 方法一
delete from s
where sno in 
(select sno from s group by sno having count(*) > 1)
and rowid not in
(select min(rowid) from s group by sno having count(*) > 1);

-- 方法二
delete from student where rowid in
(select a.rowid from student a,student b 
 where a.sno=b.sno and a.rowid > b.rowid);
 
-- 方法三
delete from student d where d.rowid >
(select min(x.rowid) from student x where d.sno=x.sno);
```
