### 目标
>理解oracle11g的临时表空间的作用   
掌握创建oracle11g的临时表空间的方法  
掌握oracle11g临时表空间的相关操作

#### oracle11g的临时表空间的作用
>临时表空间：用来存放用户的临时数据，临时数据就是
在需要时被覆盖，关闭数据库后自动删除，其中不能存放永
久性数据。
>>例如当用户对很多数据行进行排序时，排序在PGA中进
行。但是如果排序的数据过多，导致内存不足时，oracle会
把要排序的数据分成多份，每次只取一份放在PGA中进行排
序，其他的部分都放到临时表空间中，当PGA里的部分排序
完成后，把排序好的部分交换到临时表空间中，同时再从临
时表空间里取一份没有排序的数据到PGA中进行排序，这样
直到所有数据排序完成为止。  

>用户连接数据库，oracle数据库会为每个用户创建一个进程，即`用户进程1->服务器进程1PGA`。  
临时表空间在磁盘上，PGA在内存上。  
用户在进行大数据量排序时，可能要借助临时表空间。

#### oracle11g的临时表空间和临时表空间组
>临时表空间组是一组由临时表空间组成的组，临时表空
间组和临时表空间不能同名。临时表空间组不能显式地创建
和删除；当把第一个临时表空间分配给某个临时表空间组
时，会自动创建这个临时表空间组；将临时表空间组的最后
一个临时表空间删除时，会自动删除临时表空间组。

### 练习
```SQL
conn system/system@ZSR;
--查看临时文件信息：
select * from v$tempfile;

     FILE# CREATION_CHANGE# CREATION_TIME        TS#     RFILE# STATUS  ENABLED         BYTES     BLOCKS CREATE_BYTES BLOCK_SIZE NAME
---------- ---------------- ------------- ---------- ---------- ------- ---------- ---------- ---------- ------------ ---------- --------------------------------------------------------------------------------
    1           354846 2018/5/13 9:0          3          1 ONLINE  READ WRITE   20971520       2560     20971520       8192 C:\ORACLEXE\APP\ORACLE\ORADATA\XE\TEMP.DBF

--查看临时文件信息：
select * from dba_temp_files;

FILE_NAME                                                                           FILE_ID TABLESPACE_NAME                     BYTES     BLOCKS STATUS  RELATIVE_FNO AUTOEXTENSIBLE   MAXBYTES  MAXBLOCKS INCREMENT_BY USER_BYTES USER_BLOCKS
-------------------------------------------------------------------------------- ---------- ------------------------------ ---------- ---------- ------- ------------ -------------- ---------- ---------- ------------ ---------- -----------
C:\ORACLEXE\APP\ORACLE\ORADATA\XE\TEMP.DBF                                                1 TEMP                             20971520       2560 ONLINE             1 YES            3435972198    4194302           80   19922944        2432

--其他数据对应的包空间
select * from dba_data_files;

FILE_NAME                                                                           FILE_ID TABLESPACE_NAME                     BYTES     BLOCKS STATUS    RELATIVE_FNO AUTOEXTENSIBLE   MAXBYTES  MAXBLOCKS INCREMENT_BY USER_BYTES USER_BLOCKS ONLINE_STATUS
-------------------------------------------------------------------------------- ---------- ------------------------------ ---------- ---------- --------- ------------ -------------- ---------- ---------- ------------ ---------- ----------- -------------
C:\ORACLEXE\APP\ORACLE\ORADATA\XE\USERS.DBF                                               4 USERS                           104857600      12800 AVAILABLE            4 YES            1181116006    1441792         1280  103809024       12672 ONLINE
C:\ORACLEXE\APP\ORACLE\ORADATA\XE\SYSAUX.DBF                                              3 UNDOTBS1                         26214400       3200 AVAILABLE            3 YES            3435972198    4194302          640   25165824        3072 ONLINE
C:\ORACLEXE\APP\ORACLE\ORADATA\XE\UNDOTBS1.DBF                                            2 SYSAUX                          702545920      85760 AVAILABLE            2 YES            3435972198    4194302         1280  701497344       85632 ONLINE
C:\ORACLEXE\APP\ORACLE\ORADATA\XE\SYSTEM.DBF                                              1 SYSTEM                          377487360      46080 AVAILABLE            1 YES             629145600      76800         1280  376438784       45952 SYSTEM

--查看临时表空间组的信息：
select * from dba_tablespace_groups;

GROUP_NAME                     TABLESPACE_NAME
------------------------------ ------------------------------

--查看临时表空间的信息（CONTENTS=TEMPORARY）
select * from dba_tablespaces;

TABLESPACE_NAME                BLOCK_SIZE INITIAL_EXTENT NEXT_EXTENT MIN_EXTENTS MAX_EXTENTS   MAX_SIZE PCT_INCREASE MIN_EXTLEN STATUS    CONTENTS  LOGGING   FORCE_LOGGING EXTENT_MANAGEMENT ALLOCATION_TYPE PLUGGED_IN SEGMENT_SPACE_MANAGEMENT DEF_TAB_COMPRESSION RETENTION   BIGFILE PREDICATE_EVALUATION ENCRYPTED COMPRESS_FOR
------------------------------ ---------- -------------- ----------- ----------- ----------- ---------- ------------ ---------- --------- --------- --------- ------------- ----------------- --------------- ---------- ------------------------ ------------------- ----------- ------- -------------------- --------- ------------
SYSTEM                               8192          65536                       1  2147483645 2147483645                   65536 ONLINE    PERMANENT LOGGING   NO            LOCAL             SYSTEM          NO         MANUAL                   DISABLED            NOT APPLY   NO      HOST                 NO        
SYSAUX                               8192          65536                       1  2147483645 2147483645                   65536 ONLINE    PERMANENT LOGGING   NO            LOCAL             SYSTEM          NO         AUTO                     DISABLED            NOT APPLY   NO      HOST                 NO        
UNDOTBS1                             8192          65536                       1  2147483645 2147483645                   65536 ONLINE    UNDO      LOGGING   NO            LOCAL             SYSTEM          NO         MANUAL                   DISABLED            NOGUARANTEE NO      HOST                 NO        
TEMP                                 8192        1048576     1048576           1             2147483645            0    1048576 ONLINE    TEMPORARY NOLOGGING NO            LOCAL             UNIFORM         NO         MANUAL                   DISABLED            NOT APPLY   NO      HOST                 NO        
USERS                                8192          65536                       1  2147483645 2147483645                   65536 ONLINE    PERMANENT LOGGING   NO            LOCAL             SYSTEM          NO         AUTO                     DISABLED            NOT APPLY   NO      HOST                 NO        

--查找默认的临时表空间：
select property_name, property_value from 
database_properties where property_name = 
'DEFAULT_TEMP_TABLESPACE';

PROPERTY_NAME                  PROPERTY_VALUE
------------------------------ --------------------------------------------------------------------------------
DEFAULT_TEMP_TABLESPACE        TEMP

--查找默认的临时表空间结构
desc database_properties;
Name           Type           Nullable Default Comments             
-------------- -------------- -------- ------- -------------------- 
PROPERTY_NAME  VARCHAR2(30)                    Property name        
PROPERTY_VALUE VARCHAR2(4000) Y                Property value       
DESCRIPTION    VARCHAR2(4000) Y                Property description 

--创建临时表空间，不属于组
create temporary tablespace temp1 tempfile 'C:\ORACLEXE\APP\ORACLE\ORADATA\XE\TEMP1A.DBF '
size 10m autoextend on;

select *from dba_tablespaces;

--创建临时表空间，属于组temp_grp
create temporary tablespace temp2 tempfile 'C:\ORACLEXE\APP\ORACLE\ORADATA\XE\TEMP2A.DBF'
size 10m autoextend on
tablespace group temp_grp;

--select *from dba_tablespace_groups;

--把temp1加入到temp_grp组中去
alter tablespace temp1 tablespace group temp_grp;

GROUP_NAME                     TABLESPACE_NAME
------------------------------ ------------------------------
TEMP_GRP                       TEMP1
TEMP_GRP                       TEMP2

--把temp1从temp_grp组 移出去
alter tablespace temp1 tablespace group '';

--查看temp1有多少个临时表空间文件
select *from dba_temp_files;

--给temp1表空间添加 一个临时文件
alter tablespace temp1 add tempfile 'C:\ORACLEXE\APP\ORACLE\ORADATA\XE\TEMP1B.DBF '
size 10m autoextend on;

--修改系统默认的临时表空间 为 一个组
alter database default temporary tablespace temp_grp;

--查看临时表空间

--修改系统默认的临时表空间 为 一个 临时表空间
```
