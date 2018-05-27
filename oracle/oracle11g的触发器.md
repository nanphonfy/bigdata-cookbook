### 触发器
>
    触发器是当特定事件出现时自动执行的存储过程;
    特定事件可以是执行更新的DML语句和DDL语句;
    触发器不能被显式调用;
    触发器的功能：
        ①自动生成数据;
        ②自定义复杂的安全权限;
        ③提供审计和日志记录;
        ④启用复杂的业务逻辑.

#### 触发器语法
```
CREATE [OR REPLACE] TRIGGER trigger_name
AFTER | BEFORE | INSTEAD OF
[INSERT] [[OR] UPDATE [OF column_list]] 
[[OR] DELETE]
ON table_or_view_name
[REFERENCING {OLD [AS] old / NEW [AS] new}]
[FOR EACH ROW]
[WHEN (condition)]
pl/sql_block;
```

#### 组成部分
>
    触发器由三部分组成：
    触发器语句（事件）
        定义激活触发器的 DML 事件和 DDL 事件.
    触发器限制
        执行触发器的条件，该条件必须为真才能激活触发器
    触发器操作（主体）
        包含一些 SQL 语句和代码，它们在发出了触发器语句且触发限制的值为真时运行

#### 触发器操作
当用户插入、更新成绩表中的记录时候，就输出一个提示“触发器响应了”。

```
--针对整张表的
create or replace trigger trig1 before insert or update on grade
begin 
  dbms_output.put_line('触发器 trig1 响应了');
end;
/

--针对每一行的
create or replace trigger trig1 before insert or update on grade for each row
begin 
  dbms_output.put_line('触发器 trig1 响应了');
end;
/
```

有了for each row，就是行级触发器，没有for each row就是表级触发器。行级触发器，对于每一行都做触发器的动作。

:new:用户即将插入的记录；  
:old:用户即将删除的记录。

触发器不能使用：rollback、commit、create、drop、alter、saveopint等内容。

#### after和before触发器  
**after触发器工作原理：** `更新->表->保存更新->oracle数据库->激活->触发器`  
**before触发器工作原理：** `更新->表->激活->触发器->保存更新->oracle数据库`

保证学生的sno列，不能为负值，我们用触发器解决。  
当用户插入记录到student时，不能插入sno是负值的记录。  

```
create or replace trigger trig2 before insert on student for each row
begin 
  if :new.sno < 0 then
    raise_application_error(-20001,'学号错误，不能插入表中');
  end if;
  dbms_output.put_line('触发器 trig1 响应了');
end;
/
```
### 触发器类型

触发器类型->模式（DDL）触发器、数据库级触发器、DML触发器。
DML触发器包括：行级、语句级、istead of触发器


类型 | 备注
---|---
DDL触发器	|在模式中执行DDL语句时执行
数据库级触发器	|在发生打开、关闭、登录和退出数据库等系统事件时执行
DML 触发器	|在对表或视图执行DML语句时执行
语句级触发器	|无论受影响的行数是多少，都只执行一次
行级触发器	|对DML语句修改的每个行执行一次
INSTEAD OF 触发器 |用于用户不能直接使用 DML 语句修改的视图

>行级触发器：有for each row语句，在begin代码
段中可以使用:new和:old；  
>语句级触发器：没有有for each row语句，在begin代码段中不可以使用:new和:old。

>如果在触发器的plsql内使用:new :old，就必须是行级触发器，就是要有for each row。  
>>当执行insert的时候，:new存在，:old没有；  
当执行delete的时候，:new不存在,:old存在；  
当执行update的时候，:new存在，:old存在。

oracle要更新某行时，是先删除原来的记录，然后插入新的记录。

平时用的最多的是DML触发器。

### 系统常用变量

变量 | 备注
---|---
Ora_client_ip_address|返回客户端的ip地址
Ora_database_name |返回当前数据库名
Ora_login_user |返回登录用户名
Ora_dict_obj_name |返回ddl操作所对应的数据库对象名 
Ora_dict_obj_type |返回ddl操作所对应的数据库对象的类型 

#### 启用、禁用和删除触发器
- 启动和禁用触发器
```
ALTER TRIGGER aiu_itemfile DISABLE;
ALTER TRIGGER aiu_itemfile ENABLE;
```
- 删除触发器
```
DROP TRIGGER aiu_itemfile;
```

- 更新视图的触发器：
```
update view_stu_add set zz='安阳' where sname = 'Kite';

create or replace trigger tri_view instead of update on view_stu_add for each row 
declare
  aa number := 0;
begin 
  select sno into aa from student where sname = :old.sname;
  delete address where sno = aa;
  insert into address values(aa,:new.zz);
end;
/
```

- 当用户对成绩表进行增删改时，把当时的情况输出
```
create or replace trigger trig3 before insert or update or delete on grade for each row
begin
  if inserting then
    dbms_output.put_line('插入学生学号:'||:new.code||'，姓名:'||:new.name||',成绩:'||:new.score);
  end if;
  if updating then
    dbms_output.put_line('原学生学号:'||:old.code||'，姓名:'||:old.name||',成绩:'||:old.score||'，新学生学号:'||:new.code||'，姓名:'||:new.name||',成绩:'||:new.score);
  end if;
  if deleting then
    dbms_output.put_line('删除的学生学号:'||:old.code||'，姓名:'||:old.name||',成绩:'||:old.score);
  end if; 
end
/
```

- 模式触发器是一种特殊的触发器。

只要满足该模式，就会在我们的模式触发器表，生成记录。


- 数据库启动、关闭触发器

```
create table event_table（event varchar2(3),time date);

create or replace trigger tr_startup
after startup on database
begin 
  insert into event_table values(ora_sysevent,sysdate);
end;
/

create or replace trigger tr_shutdown
before shutdown on database
begin 
  insert into event_table values(ora_sysevent,sysdate);
end;
/

select event,to_char(time,'yyyymmdd hh24:mi:ss') from event_table;
```


>关闭数据库  
--shutdown immediate   
启动数据库  
--startup

- 用户登录、退出触发器

```
create table event_table（username varchar2(3),logon_time date,logoff_time date,address varchar2(20));

create or replace trigger tr_logon
after logon on database
begin 
  insert into log_table values(ora_login_user,sysdate,ora_client_ip_address);
end;
/

create or replace trigger tr_logoff
before logoff on database
begin 
  insert into log_table values(ora_login_user,sysdate,ora_client_ip_address);
end;
/

--显示用户  
show user;
```