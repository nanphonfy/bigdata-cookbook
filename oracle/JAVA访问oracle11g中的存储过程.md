### 目标
>
    掌握JAVA中调用oracle11g的几种不同类型的存储过程的方法，即：没有返回参数的过程(插入记录和更新记录两个例子)、有返回参数的过程、返回列表的过程、返回带分页的列表的过程。

>D:\app\Administrator\product\11.1.0\db_1\jdbc\lib\下的ojdbc6.jar文件

- [Maven导入ojdbc6.jar](https://blog.csdn.net/qq_31776219/article/details/53669203)  

- 使用system/system登录，并查看当前数据库名称。  
[select * from v$database;](https://blog.csdn.net/jerry_fight/article/details/7911263)


```
//jar包ojdbc6.jar
String driver = "oracle.jdbc.driver.OracleDriver";
String url = "jdbc:oracle:thin:@192.168.25.146:1521:XE";
```

```
--没有返回参数的存储过程
create or replace procedure test_no_out_param(no in int,name in varchar2,age in int)
is
begin
  insert into student(sno,sname,sage) values(no,name,age);
  commit;
end;
/
--更新年龄
create or replace procedure test_no_out_param_update(no in int,age in int)
is
begin
  update student set sage = age where sno = no;
  commit;
end;
/
```

```
--有返回参数的存储过程
create or replace procedure test_out_param(no in int,name out varchar2,age out int)
is
begin
  select sname,sage into name,age from student where sno = no;
end;
/


--存储过程返回列表
create or replace package outCursorPack
is
  type outCursor is ref cursor;
end outCursorPack;
/

create or replace procedure test_out_result_set(p_cursor out outCursorPack.outCursor)
is
begin
  open p_cursor for select *from student;
end;
/

-- 实现分页的存储过程
-- ps 每页几个记录，cs 显示第几页，
create or replace procedure test_page(ps int,cs int,p_cursor out outCursorPack.outCursor)
is
begin
  open p_cursor for 
  select *from (select s.*,rownum rn from student s) where rn>ps*(cs-1) and
  rn <= ps*cs order by sno;
end;
/
```

### 使用java编码

```
import java.sql.*;

/**
 * @author nanphonfy(南风zsr)
 * @date 2018/6/3
 */
public class OracleProcedureTest {
    String driver = "oracle.jdbc.driver.OracleDriver";
    String url = "jdbc:oracle:thin:@192.168.25.146:1521:XE";
    ResultSet rs = null;
    Connection conn = null;
    CallableStatement cstmt = null;

    public static void main(String[] args) {
        OracleProcedureTest oracleProcedureTest = new OracleProcedureTest();
//        oracleProcedureTest.testNoOutParameterInsert(8, "革命", 35);
//        oracleProcedureTest.testNoOutParameterUpdate(6, 34);
//        oracleProcedureTest.testOutParameter(6);
        //oracleProcedureTest.testOutResultSet();
        oracleProcedureTest.testPage(4,2);
    }

    /**
     * 没有返回参数的存储过程
     * @param no
     * @param name
     * @param age
     */
    public void testNoOutParameterInsert(int no, String name, int age) {
        try {
            Class.forName(driver);
            conn = DriverManager.getConnection(url, "zsr", "zsr");
            cstmt = conn.prepareCall("{ call zsr.test_no_out_param(?,?,?)}");

            cstmt.setInt(1, no);
            cstmt.setString(2, name);
            cstmt.setInt(3, age);

            cstmt.execute();
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }

    /**
     * 更新年龄
     * @param no
     * @param age
     */
    public void testNoOutParameterUpdate(int no, int age) {
        try {
            Class.forName(driver);
            conn = DriverManager.getConnection(url, "zsr", "zsr");
            cstmt = conn.prepareCall("{ call zsr.test_no_out_param_update(?,?)}");

            cstmt.setInt(1, no);
            cstmt.setInt(2, age);

            cstmt.execute();
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }

    /**
     * 有返回参数的存储过程
     * @param no
     */
    public void testOutParameter(int no) {
        try {
            Class.forName(driver);
            conn = DriverManager.getConnection(url, "zsr", "zsr");
            cstmt = conn.prepareCall("{ call zsr.test_out_param(?,?,?)}");
            cstmt.setInt(1, no);
            cstmt.registerOutParameter(2, Types.VARCHAR);
            cstmt.registerOutParameter(3, Types.INTEGER);
            cstmt.execute();
            String name = cstmt.getString(2);
            Integer age = cstmt.getInt(3);

            System.out.println(String.format("学号：%s,姓名：%s,年龄：%s", no, name, age));
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }

    /**
     *存储过程返回列表
     */
    public void testOutResultSet() {
        try {
            Class.forName(driver);
            conn = DriverManager.getConnection(url, "zsr", "zsr");
            cstmt = conn.prepareCall("{ call zsr.test_out_result_set(?)}");
            cstmt.registerOutParameter(1, oracle.jdbc.OracleTypes.CURSOR);
            cstmt.execute();
            rs = (ResultSet) cstmt.getObject(1);
            while (rs.next()) {
                System.out.println(String.format("学号：%s,姓名：%s,年龄：%s", rs.getInt(1), rs.getString(2), rs.getInt(3)));
            }
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }

    /**
     *  实现分页的存储过程
     ps 每页几个记录，cs 显示第几页，
     */
    public void testPage(int pageSize,int page){
        try {
            Class.forName(driver);
            conn = DriverManager.getConnection(url, "zsr", "zsr");
            cstmt = conn.prepareCall("{ call zsr.test_page(?,?,?)}");
            cstmt.setInt(1, pageSize);
            cstmt.setInt(2, page);
            cstmt.registerOutParameter(3, oracle.jdbc.OracleTypes.CURSOR);
            cstmt.execute();
            rs = (ResultSet) cstmt.getObject(3);
            while (rs.next()) {
                System.out.println(String.format("学号：%s,姓名：%s,年龄：%s", rs.getInt(1), rs.getString(2), rs.getInt(3)));
            }
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}

```
