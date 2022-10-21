# JDBC

JDBC是什么？Java DataBase Connectivity(Java语言连接数据库）

**JDBC的本质是什么？**

JDBC是SUN公司制定的一套接口（interface)

接口都有调用者和实现者。

面向接口调用、面向接口写实现类，这都属于面向接口编程。

**为什么要面向接口编程？**

解耦合：降低程序的耦合度，提高程序的扩展力。

多态机制就是非常典型的：面向接口编程。（不要面向具体编程）

建议：

```java
Animal a=new Cat（）；
Animal a=new Dog（）；
public	void	feed（Animal	a）{}
//不建议：
Dog d=new Dog（）；
Cat  c=new Cat（）；
```

**为什么SUN指定一套JDBC接口呢？**

因为每一个数据库的底层实现原理都不一样。
Oracle数据库有自己的原理。
MySQL数据库也有自己的原理。
MS SqlServer数据库也有自己的原理。

每一个数据库产品都有自己独特的实现原理。

**JDBC编程六步（需要背会）**

1. 注册驱动（告诉JVM，即将要连接的是哪个品牌的数据库）
2. 获取连接（表示JVM的进程和数据库进程之间的通道打开了）
3. 获取数据库操作对象（专门执行sql语句的对象）
4. 执行SQL语句（DQL DML...)
5. 处理查询结果集(只有当第四步执行的是select语句的时候，才有这第五步处理查询结果集）
6. 释放资源（使用完资源之后一定要关闭资源,java和数据库属于进程间的通信，开启之后一定要关闭）。

url:统一资源定位符

url包括：协议、IP、PORT、资源名

JDBC编程六步

```java
//将连接数据库的所有信息配置的配置文件中
//实际开发中不建议把连接数据库的信息写死到java程序中。
import java.sql.*;
public class JDBCTest01{
	public static void mian(String[] args){
	//1.注册驱动
	statement stmt=null;
	Connection conn=null;
	try{
	//注册驱动的第一种方式
	Driver driver=new com.mysql.jdbc.Driver();
	DriverManager.registerDriver(driver);
	//注册驱动的第二种方式，常用的
	//为什么这种方式常用？因为参数是一个字符串，字符串可以写到配置文件当中。
	//一下方法不需要接收返回值，因为我们只想用它的类加载的动作。
	Class.forNmae("com.mysql.jdbc.Driver");
	//2.获取连接
	String url="jdbc:mysql://localhost:3306/test";
	String user="root";
	String password="root";
	Connection conn=DriverManager.getConnection(url,user,password);
	//3.获取数据库操作对象（statement专门执行sql语句的）
	Statement	stmt=conn.createStatement();
	//4.执行sql
	//JDBC不需要写;号
	String sql="insert into dept(deptno,danme,loc) values(50,'人事部','北京')";
	//专门执行DML语句（insert delete update)
	//返回值是“影响数据库中的记录条数”
	intcount=stmt.executeUpdate(sql);
	System.out.println(count == 1?"保存成功"：“保存失败”);
	//5、处理查询结果集
	}catch(SQLException e){
		e.printStackTrace();
	}finally{
	//6、释放资源,为了保证资源一定释放，在finally语句块中关闭资源
	//并且要遵循从小到大依次关闭
	//分别对其try..catch
	try{
	if(stmt!=null){
	stmt.close();
	}catch(SQLException e){
	e.printStackTrace();
	}
	if(conn!=null){
	conn.close();
	}
	}catch(SQLException e){
	e.printStackTrace();
	}
}
}
}
```

**JDBC中关闭事务自动提交机制**
conn.setAutoCommit(false);获取数据库连接之后就可以设置
conn.commit();在程序执行结束之后没有异常使用该语句
conn.rollback();该语句使用在程序的异常捕捉处理处