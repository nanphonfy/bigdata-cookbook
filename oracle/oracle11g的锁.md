### 目标
>- 理解锁定的概念
>- 掌握select  for update的使用
>- 理解不同的锁定的含义

#### 锁的概念
> 锁是数据库用来控制共享资源并发访问的机制；  
锁用于保护正在被修改的数据；  
直到提交或回滚了事务之后，其他用户才可以更新数据。

#### 锁定的优点
>- 一致性 - 一次只允许一个用户修改数据
>- 完整性 - 为所有用户提供正确的数据。如果一个用户进行了修改并保存，所做的修改将反映给所有用户  
>- 并行性 - 允许多个用户访问同一数据

锁的类型：行级锁、表级锁

#### 行级锁
>
    对正在被修改的行进行锁定。其他用户可以访问除被锁定的行以外的行  
    行级锁是一种排他锁，防止其他事务修改此行  
    在使用以下语句时，Oracle会自动应用行级锁：  
        INSERT
        UPDATE
        DELETE
        SELECT … FOR UPDATE
    SELECT … FOR UPDATE语句允许用户一次锁定多条记录进行更新
    使用COMMIT或ROLLBACK语句释放锁

- SELECT … FOR UPDATE语法:
>
    　SELECT … FOR UPDATE [OF   columns] [WAIT n | NOWAIT];

```SQL
SQL> SELECT * FROM emp WHERE sal=1000
    FOR UPDATE;
SQL> UPDATE emp SET sal = 3000
    WHERE  sal =1000;
SQL> COMMIT;

SQL> SELECT * FROM scott.emp WHERE sal=1000
    FOR UPDATE WAIT 5;

SQL> SELECT * FROM scott.emp WHERE sal=1000
    FOR UPDATE NOWAIT ;
```

#### 表级锁
锁定整个表，限制其他用户对表的访问

```SQL
SQL> select * from grade;

       SNO KM              SCORE GRADE
---------- ---------- ---------- ------
         1 语文               65 
         2 数学               76 
         3 英语               86 
         4 语文               94 

--zsr用户，未提交事务前，未释放锁
SQL> update grade g set g.score = 80
	 where g.sno = 4;
	 
--system用户，执行不了，等待
SQL> update zsr.grade g set g.score = 85
	 where g.sno = 4;
	 
system/system/
--查看锁
SQL> select *from v$lock;


select *from dba_objects do
where do.object_id = '20288';

SQL> select *from v$lock;

ADDR     KADDR           SID TYPE        ID1        ID2      LMODE    REQUEST      CTIME      BLOCK
-------- -------- ---------- ---- ---------- ---------- ---------- ---------- ---------- ----------
359949BC 359949E8          5 PW            1          0          3          0    1476140          0
35994E94 35994EC0          5 MR            4          0          4          0    1476142          0
35994E18 35994E44          5 MR            3          0          4          0    1476142          0
35994D20 35994D4C          5 MR            1          0          4          0    1476142          0
35994D9C 35994DC8          5 MR            2          0          4          0    1476142          0
35994F10 35994F3C          5 MR          201          0          4          0    1476141          0
35994A38 35994A64          6 RS           25          1          2          0    1476153          0
35994848 35994874          6 XR            4          0          1          0    1476158          0
359948C4 359948F0          6 RD            1          0          1          0    1476158          0
35994940 3599496C          6 CF            0          0          2          0    1476158          0
35994BAC 35994BD8         12 AE          100          0          4          0        394          0
35995084 359950B0         22 AE          100          0          4          0        190          0
35994CA4 35994CD0         90 KT            1          0          4          0    1476134          0
35994B30 35994B5C         92 RT            1          0          6          0    1476154          0
35995008 35995034         93 TS            3          1          3          0    1476135          0
35994C28 35994C54         94 AE          100          0          4          0    1476129          0
35994F8C 35994FB8         97 AE          100          0          4          0        259          0
34193BD0 34193C10         97 TX       262149        367          6          0        211          0
3703A688 3703A6B8         97 TM        20288          0          3          0        210          0
35995100 3599512C        104 AE          100          0          4          0         24          0
20 rows selected

--TM表级锁，TX行级锁
--ID1表的编号



SQL> 
SQL> select *from dba_objects do
  2  where do.object_id = '20288';
Warning: connection was lost and re-established

OWNER                          OBJECT_NAME                                                                      SUBOBJECT_NAME                  OBJECT_ID DATA_OBJECT_ID OBJECT_TYPE         CREATED     LAST_DDL_TIME TIMESTAMP           STATUS  TEMPORARY GENERATED SECONDARY  NAMESPACE EDITION_NAME
------------------------------ -------------------------------------------------------------------------------- ------------------------------ ---------- -------------- ------------------- ----------- ------------- ------------------- ------- --------- --------- --------- ---------- ------------------------------
ZSR                            GRADE                                                                                                                20288          20288 TABLE               2018/6/10 1 2018/6/10 17: 2018-06-10:17:51:28 VALID   N         N         N                  1 

--解锁：rollback

select *from grade g 
where g.sno = 4 for update;




```
