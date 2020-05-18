## JDBC的本质

JDBC是什么？
Java DataBase Connectivity（Java语言连接数据库）

JDBC的本质是什么？
JDBC是SUN公司制定的一套接口（interface）
接口都有调用者和实现者。
面向接口调用、面向接口写实现类，这都属于面向接口编程。

为什么要面向接口编程？
解耦合：降低程序的耦合度，提高程序的拓展力。
多态机制就是非常典型的面向抽象编程。

## JDBC编程

JDBC编程六步
第一步：注册驱动（作用：告诉Java程序即将要连接的是哪个数据库）
第二步：获取连接（表示JVM的进程和数据库进程之间的通道打开了，这属于进程间的通信，使用完一定要关闭）
第三步：获取数据库操作对象（专门执行sql语句的对象）
第四步：执行SQL语句（DQL DML）
第五步：处理查询结果集（只有当第四步执行的是select语句的时候才会有第五步处理查询结果集）
第六步：释放资源（使用完资源之后一定要关闭资源。Java和数据库属于进程间的通信，开启之后一定要关闭。）

JDBC插入、删除、更新数据演示：

~~~java
import java.sql.*;

public class JDBCTest01 {
	public static void main(String[] args) {
		Connection conn = null;
		Statement stmt = null;
		try 
		{
			//1.注册驱动
			DriverManager.registerDriver(new com.mysql.cj.jdbc.Driver());
			
			//2.获取连接
			String url = "jdbc:mysql://localhost:3306/bjpowernode?
                                                 serverTimezone=UTC&useSSL=false";
			String user = "root";
			String password = "password";
			conn = DriverManager.getConnection(url,user,password);
			System.out.println("数据库连接对象="+conn);
			
			//3.获取数据库操作对象(Statement专门执行sql语句)
			stmt = conn.createStatement();
			
			//4.执行sql 插入
			String sql = "insert into dept(deptno,dname,loc) values(50,'人事部','北京')";
            
            //删除
            //String sql = "delete from dept where deptno = 40";
            //更新
            //String sql = "update dept set dname = '销售部',loc = '天津' where deptno = 20";
            
			//专门执行DML语句的（insert delete update）
			//返回值是“影响数据库中的记录条数”
			int count = stmt.executeUpdate(sql);
			System.out.println(count==1 ? "保存成功" : "保存失败");		
		}
		catch(SQLException e)
		{
			e.printStackTrace();
		}
		finally
		{
			//6.释放资源
			//为了保证资源一定释放，在finally语句块中关闭资源
			//并且要遵循从小到大依次关闭
			//分别对其try..catch
			try
			{
				if(stmt!=null)
					stmt.close();
			}
			catch(SQLException e)
			{
				e.printStackTrace();
			}
			
			try
			{
				if(conn!=null)
					conn.close();
			}
			catch(SQLException e)
			{
				e.printStackTrace();
			}			
		}
	}
}
~~~

## JDBC注册驱动的两种方式

~~~Java
import java.sql.*;

public class JDBCTest02 {
	public static void main(String[] args) {

		try 
		{
			//1.注册驱动
             //这是注册驱动的第一种方式
			//DriverManager.registerDriver(new com.mysql.cj.jdbc.Driver());
            
            //注册驱动的第二种方式（常用的）
            //为什么这种方式常用？因为参数是一个字符串，字符串可以写到xxx.propertites文件中。
            //以下方式不需要接受返回值，因为我们只想用它的类加载动作。
            Class.forName("com.mysql.cj.jdbc.Driver");
			
			//2.获取连接
			String url = "jdbc:mysql://localhost:3306/bjpowernode?
                                                 serverTimezone=UTC&useSSL=false";
			String user = "root";
			String password = "password";
			Connection conn = DriverManager.getConnection(url,user,password);
			System.out.println("数据库连接对象="+conn);
		}
		catch(SQLException e)
		{
			e.printStackTrace();
		}
        catch(ClassNotFoundException e)
		{
			e.printStackTrace();
		}
	
	}
}
~~~

## 从属性资源文件中读取连接数据库的信息

~~~java
package jabc_connect;
import java.sql.*;
import java.util.*;

//将数据库的所有信息配置到配置文件中
public class JDBCTest02 {
    public static void main(String[] args) {

        //使用资源绑定器绑定属性配置文件
        ResourceBundle bundle = ResourceBundle.getBundle("jdbc");
        String driver = bundle.getString("driver");
        String url = bundle.getString("url");
        String user = bundle.getString("user");
        String password = bundle.getString("password");

        Connection conn = null;
        Statement stmt = null;
        try
        {
            //1.注册驱动
            Class.forName(driver);

            //2.获取连接
            conn = DriverManager.getConnection(url,user,password);
            System.out.println("数据库连接对象="+conn);

            //3.获取数据库操作对象(Statement专门执行sql语句)
            stmt = conn.createStatement();

            //4.执行sql 插入
            String sql = "insert into dept(deptno,dname,loc) values(50,'人事部','北京')";

            int count = stmt.executeUpdate(sql);
            System.out.println(count==1 ? "保存成功" : "保存失败");
        }
        catch(Exception e)
        {
            e.printStackTrace();
        }
        finally
        {
            //6.释放资源
            //为了保证资源一定释放，在finally语句块中关闭资源
            //并且要遵循从小到大依次关闭
            //分别对其try..catch
            try
            {
                if(stmt!=null)
                    stmt.close();
            }
            catch(SQLException e)
            {
                e.printStackTrace();
            }

            try
            {
                if(conn!=null)
                    conn.close();
            }
            catch(SQLException e)
            {
                e.printStackTrace();
            }
        }
    }
}
~~~

~~~properties
driver = com.mysql.cj.jdbc.Driver
url = jdbc:mysql://localhost:3306/bjpowernode?serverTimezone=UTC&useSSL=false
user = root
password = password
~~~

## 处理查询结果集

~~~Java
package jabc_connect;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.Statement;

public class JDBC_select
{
    public static void main(String[] args)
    {
        Connection conn = null;
        Statement stmt = null;
        ResultSet rs = null;

        try
        {
            Class.forName("com.mysql.cj.jdbc.Driver");

            String url = "jdbc:mysql://localhost:3306/bjpowernode?      
                                                   serverTimezone=UTC&useSSL=false";
            String user = "root";
            String password = "password";
            conn = DriverManager.getConnection(url,user,password);

            stmt = conn.createStatement();

            String sql = "select empno,ename,sal from emp";
            rs = stmt.executeQuery(sql);  //专门执行DQL语句的方法

            //处理查询结果集

            /*
            处理一行数据
            boolean flag = rs.next();
            if(flag)
            {
                //光标指向的行有数据，取数据
                //getString()方法的特点是：不管数据库中的数据类型是什么，都以String的形式取出。
                String empno = rs.getString(1);//JDBC中所有下标从一开始，不从0开始。
                String ename = rs.getString(2);
                String sal = rs.getString(3);
                System.out.println(empno + "," + ename + "," + sal);
            }
            */

            while(rs.next())
            {
                /*
                String empno = rs.getString(1);
                String ename = rs.getString(2);
                String sal = rs.getString(3);
                System.out.println(empno + "," + ename + "," + sal);
                */

                String empno = rs.getString("empno");
                String ename = rs.getString("ename");
                String sal = rs.getString("sal");
                System.out.println(empno + "," + ename + "," + sal);
            }

        }
        catch(Exception e)
        {
            e.printStackTrace();
        }
        finally
        {
            if(rs!=null)
            {
                try
                {
                    rs.close();
                }
                catch(Exception e)
                {
                    e.printStackTrace();
                }
            }

            if(stmt!=null)
            {
                try
                {
                    stmt.close();
                }
                catch(Exception e)
                {
                    e.printStackTrace();
                }
            }

            if(conn!=null)
            {
                try
                {
                    conn.close();
                }
                catch(Exception e)
                {
                    e.printStackTrace();
                }
            }

        }
    }
}
~~~

## 模拟用户登录

需求：模拟用户登录功能的实现

业务描述：程序运行的时候，提供一个输入的入口，可以让用户输入用户名和密码。
                  用户输入用户名和密码之后，提交信息，Java程序收集用户信息。
		  Java程序连接数据库验证用户名和密码是否合法。
		  合法：显示登陆成功。
		  不合法：显示登陆失败。

~~~Java
package jabc_connect;

import java.sql.*;
import java.util.*；

public class JDBC_login {
    public static void main(String[] args)
    {
        //初始化一个界面
        Map<String,String> userLoginInfo = initUI();
        //验证用户名和密码
        boolean loginSuccess = login(userLoginInfo);
        //最后输出结果
        System.out.println(loginSuccess ? "登录成功" : "登录失败");
    }

    /**
     * 用户登录
     * @param userLoginInfo 用户登录信息
     * @return false表示失败，true表示成功
     */
    private static boolean login(Map<String, String> userLoginInfo) {
        //打标记
        boolean loginSuccess = false;
        //JDBC代码
        String loginName = userLoginInfo.get("loginName");
        String loginPwd = userLoginInfo.get("loginPwd");

        Connection conn = null;
        Statement stmt = null;
        ResultSet rs = null;

       try
       {
           Class.forName("com.mysql.cj.jdbc.Driver");
           String url = "jdbc:mysql://localhost:3306/bjpowernode?
                                                        serverTimezone=UTC&useSSL=false";
           String user = "root";
           String password = "password";
           conn = DriverManager.getConnection(url,user,password);

           stmt = conn.createStatement();

           String sql = "select * from t_user where loginName = 
                                             '"+loginName+"' and loginPwd = '"+loginPwd+"'";
           rs =  stmt.executeQuery(sql);

           if(rs.next())
           {
                loginSuccess = true;
           }

       }
       catch (Exception e)
       {
            e.printStackTrace();
       }
       finally
       {
           if(rs!=null)
           {
               try
               {
                   rs.close();
               }
               catch (SQLException e)
               {
                   e.printStackTrace();
               }
           }

           if(stmt!=null)
           {
               try
               {
                   stmt.close();
               }
               catch (SQLException e)
               {
                   e.printStackTrace();
               }
           }

           if(conn!=null)
           {
               try
               {
                   conn.close();
               }
               catch (SQLException e)
               {
                   e.printStackTrace();
               }
           }

       }
       return loginSuccess;
    }

    /**
     * 初始化用户界面
     * @return 用户输入的用户名和密码等登录信息
     */
    private static Map<String, String> initUI() {
        Scanner s = new Scanner(System.in);
        System.out.print("用户名：");
        String loginName = s.nextLine();
        System.out.print("密码：");
        String loginPwd = s.nextLine();
        Map<String,String> userLoginInfo = new HashMap<>();
        userLoginInfo.put("loginName",loginName);
        userLoginInfo.put("loginPwd",loginPwd);
        return userLoginInfo;
    }
}
~~~

## SQL注入现象

上面程序存在的问题：

用户名：fdsa
密码：fdsa' or '1' ='1
登录成功
这种现象被称为SQL注入（安全隐患）。

导致SQL注入的根本原因是什么？
用户输入的信息中含有sql语句的关键字，并且这些关键字参与sql语句的编译过程，
导致sql语句的原意被扭曲，进而达到sql注入。

如何解决sql注入问题？
只要用户提供的信息不参与sql语句的编译过程，问题就解决了。
关键：即使用户提供的信息中含有sql语句的关键字，但是没有参与编译，不起作用。
要想用户信息不参与sql语句的编译，那么必须使用java.sql.PreparedStatement 
PreparedStatement接口继承了java.sql.Statement  
PreparedStatement是属于预编译的数据库操作对象。
PreparedStatement的原理是：预先对SQL语句的框架进行编译，然后再给SQL语句传“值”。

~~~java
package jabc_connect;

import java.sql.*;
import java.util.*;

public class JDBC_login_plus {
    public static void main(String[] args)
    {
        //初始化一个界面
        Map<String,String> userLoginInfo = initUI();
        //验证用户名和密码
        boolean loginSuccess = login(userLoginInfo);
        //最后输出结果
        System.out.println(loginSuccess ? "登录成功" : "登录失败");
    }

    /**
     * 用户登录
     * @param userLoginInfo 用户登录信息
     * @return false表示失败，true表示成功
     */
    private static boolean login(Map<String, String> userLoginInfo){
        //打标记
        boolean loginSuccess = false;
        //JDBC代码
        String loginName = userLoginInfo.get("loginName");
        String loginPwd = userLoginInfo.get("loginPwd");

        Connection conn = null;
        PreparedStatement ps = null;
        ResultSet rs = null;

        try
        {
            //1.注册驱动
            Class.forName("com.mysql.cj.jdbc.Driver");
            //2.获取连接
            String url = "jdbc:mysql://localhost:3306/bjpowernode?
                                                             serverTimezone=UTC&useSSL=false";
            String user = "root";
            String password = "password";
            conn = DriverManager.getConnection(url,user,password);
            //3.获取预编译的数据库操作对象
            //sql语句的框架。其中，一个？表示一个占位符，将来接受一个“值”。
            //注意：占位符不能使用单引号括起来。
            String sql = "select * from t_user where loginName = ? and loginPwd = ?";
            //程序执行到此处，会发送sql语句框给DBMS，然后DBMS进行sql语句的预先编译。
            ps = conn.prepareStatement(sql);
            //给占位符？传值（第一个问号下标是1，第二个问号下标是2，JDBC中所有下标从一开始。）
            ps.setString(1,loginName);
            ps.setString(2,loginPwd);
            //4.执行sql
            rs =  ps.executeQuery();
            //5.处理结果集
            if(rs.next())
            {
                loginSuccess = true;
            }

        }
        catch (Exception e)
        {
            e.printStackTrace();
        }
        finally
        {
            //6.释放资源
            if(rs!=null)
            {
                try
                {
                    rs.close();
                }
                catch (SQLException e)
                {
                    e.printStackTrace();
                }
            }

            if(ps!=null)
            {
                try
                {
                    ps.close();
                }
                catch (SQLException e)
                {
                    e.printStackTrace();
                }
            }

            if(conn!=null)
            {
                try
                {
                    conn.close();
                }
                catch (SQLException e)
                {
                    e.printStackTrace();
                }
            }

        }
        return loginSuccess;
    }

    /**
     * 初始化用户界面
     * @return 用户输入的用户名和密码等登录信息
     */
    private static Map<String, String> initUI() {
        Scanner s = new Scanner(System.in);
        System.out.print("用户名：");
        String loginName = s.nextLine();
        System.out.print("密码：");
        String loginPwd = s.nextLine();
        Map<String,String> userLoginInfo = new HashMap<>();
        userLoginInfo.put("loginName",loginName);
        userLoginInfo.put("loginPwd",loginPwd);
        return userLoginInfo;
    }
}
~~~

Statement和PreparedStatement的区别？

Statement存在注入问题，PreparedStatement解决了SQL注入问题。
Statement是编译一次，执行一次。PreparedStatement是编译一次，执行N次。PreparedStatement效率更高。
PreparedStatement会在编译阶段做类型的安全检查。

综上所述，PreparedStatement使用较多。只有极少数情况下需要使用Statement。
什么情况下必须使用Statement呢？
业务方面要求必须支持SQL注入的时候。
Statement支持SQL注入，凡是业务方面要求是需要进行sql语句拼接的，必须使用Statement。

## PreparedStatement完成增删改

~~~Java
package jabc_connect;

import java.sql.*;
//PreparedStatement完成INSERT DELETE UPDETE

public class JDBC_PreparedStatement
{
    public static void main(String[] args)
    {
        Connection conn = null;
        PreparedStatement ps = null;
        try
        {
            Class.forName("com.mysql.cj.jdbc.Driver");

            String url = "jdbc:mysql://localhost:3306/bjpowernode?serverTimezone=UTC&useSSL=false";
            String user = "root";
            String password = "password";
            conn = DriverManager.getConnection(url, user, password);

           /* String sql = "insert into dept(deptno,dname,loc) values(?,?,?) ";
            ps = conn.prepareStatement(sql);
            ps.setInt(1, 60);
            ps.setString(2, "销售部");
            ps.setString(3, "上海");*/

          /* String sql = "update dept set dname = ?,loc = ? where deptno = ?";
           ps = conn.prepareStatement(sql);
           ps.setString(1,"研发一部");
           ps.setString(2,"北京");
           ps.setInt(3,60);*/

          String sql = "delete from dept where deptno = ?";
          ps = conn.prepareStatement(sql);
          ps.setInt(1,60);

            int count = ps.executeUpdate();
            System.out.println(count);
        }
        catch(Exception e)
        {
            e.printStackTrace();
        }
        finally
        {
            if (ps != null)
            {
                try
                {
                    ps.close();
                }
                catch (SQLException e)
                {
                    e.printStackTrace();
                }

                if (conn != null)
                    try {
                        conn.close();
                    } catch (SQLException e) {
                        e.printStackTrace();
                    }

            }
        }
    }
}
~~~

## JDBC中的事务机制

JDBC中的事务是自动提交的，什么是自动提交？
只要执行任意一条DML语句，则会自动提交一次，这是JDBC默认的事务行为。
在实际的业务当中，通常都是N条DML语句共同联合才能完成的，必须保证这些DML语句在同一个事务中同时成功或者同时失败。

账户转账演示事务
重点有三行代码：
	conn.setAutoCommit(false);   //将自动提交修改为手动提交
	conn.commit();  //提交事务
	conn.rollback();  //回滚事务

~~~Java
package jabc_connect;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.SQLException;

public class JDBC_transaction {
    public static void main(String[] args)
    {
        Connection conn = null;
        PreparedStatement ps = null;
        try
        {
            Class.forName("com.mysql.cj.jdbc.Driver");

            String url = "jdbc:mysql://localhost:3306/bjpowernode?
                                                           serverTimezone=UTC&useSSL=false";
            String user = "root";
            String password = "password";
            conn = DriverManager.getConnection(url, user, password);

            //将自动提交修改为手动提交
            conn.setAutoCommit(false);   //开启事务

            String sql = "update t_act set balance = ? where actno = ?";
            ps = conn.prepareStatement(sql);
            ps.setDouble(1,10000);
            ps.setInt(2,111);
            int count = ps.executeUpdate();

            String s = null;
            s.toString();

            ps.setDouble(1,10000);
            ps.setInt(2,222);
            count += ps.executeUpdate();

            System.out.println(count == 2 ? "转账成功": "转账失败");

            //程序能够运行到这，说明以上程序没有异常，事务结束，手动提交数据
            conn.commit();  //提交事务
        }
        catch(Exception e)
        {
            if(conn!=null)
            {
                try
                {
                    conn.rollback();  //回滚事务
                }
                catch(SQLException e1)
                {
                    e1.printStackTrace();
                }
            }
            e.printStackTrace();
        }
        finally
        {
            if (ps != null)
            {
                try
                {
                    ps.close();
                }
                catch (SQLException e)
                {
                    e.printStackTrace();
                }

                if (conn != null)
                    try {
                        conn.close();
                    } catch (SQLException e) {
                        e.printStackTrace();
                    }
            }
        }
    }
}
~~~

## JDBC工具类的封装

~~~java
package jabc_connect.utils;

import java.sql.*;

/**
 * JDBC工具类，简化JDBC编程
 */

public class DBUtil {
    /**
     * 工具类中的构造方法都是私有的。
     * 因为工具类当中的方法都是静态的，不需要new对象，直接采用类名调用。
     */
    private DBUtil(){}

    //静态代码块在类加载时执行，并且只执行一次。
    static {
        try {
            Class.forName("com.mysql.cj.jdbc.Driver");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    /**
     * 获取数据库连接对象
     * @return 连接对象
     * @throws SQLException
     */
    public static Connection getConnection() throws SQLException {
        String url = "jdbc:mysql://localhost:3306/bjpowernode?serverTimezone=UTC&useSSL=false";
        String user = "root";
        String password = "password";
        Connection conn = DriverManager.getConnection(url, user, password);
        return conn;
    }

    /**
     * 关闭资源
     * @param conn 连接对象
     * @param ps 数据库操作对象
     * @param rs 结果集
     */
    public static void close(Connection conn, Statement ps, ResultSet rs){
        if (rs!=null) {
            try{
                rs.close();
            }catch (SQLException e){
                e.printStackTrace();
            }
        }

        if (ps!=null) {
            try{
                ps.close();
            }catch (SQLException e){
                e.printStackTrace();
            }
        }

        if (conn!=null) {
            try{
                conn.close();
            }catch (SQLException e){
                e.printStackTrace();
            }
        }
    }
}
~~~

## JDBC实现模糊查询

~~~java
package jabc_connect;

import jabc_connect.utils.DBUtil;

import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;

/**
 * 这个程序有两个任务：
 *      第一，测试上面的DBUtil是否好用
 *      第二，模糊查询怎么写？
 */

public class JDBC_Util {
    public static void main(String[] args){
        Connection conn = null;
        PreparedStatement ps = null;
        ResultSet rs = null;
        try {
            //获取连接
            conn = DBUtil.getConnection();
            //获取预编译的数据库操作对象
            String sql = "select ename from emp where ename like ?";
            ps = conn.prepareStatement(sql);
            ps.setString(1,"_A%");
            rs = ps.executeQuery();
            while(rs.next()){
                System.out.println(rs.getString("ename"));
            }
        } catch (Exception e) {
            e.printStackTrace();
        }finally {
            //释放资源
            DBUtil.close(conn,ps,rs);
        }
    }
}
~~~

## 悲观锁与乐观锁

悲观锁：事务必须排队执行，数据锁住了，不允许并发。
		（行级锁：select后面添加for update）

乐观锁：支持并发，事务也不需要排队，只不过需要一个版本号。