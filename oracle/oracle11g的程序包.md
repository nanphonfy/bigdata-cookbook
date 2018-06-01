### 程序包
>①程序包是对相关过程、函数、变量、游标和异常等对象的封装；  
②程序包由规范和主体两部分组成。

- 程序包：规范+主体  
**规范：** 声明程序包中公共对象。包括类型、变量、常量、异常、游标规范和子程序规范等；  
**主体：** 声明程序包私有对象和实现在包规范中声明的子程序和游标。

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

