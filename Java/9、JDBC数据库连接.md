## JDBC数据库连接

#### 1、什么是JDBC：

JDBC（Java DataBase Connectivity，java数据库连接）是一种用于执行SQL语句的Java API，可以为多种关系数据库提供统一访问，它由一组用Java语言编写的类和接口组成。

在应用程序中并不能直接使用数据库，必须通过相应的数据库驱动程序进行数据库连接，这些驱动由数据库厂商提供。

#### 2、MySQL数据库连接：

- **添加驱动包或依赖：**

  要在程序中连接数据库必须先导入数据库连接的驱动包 `mysql-connector-java-5.1.47.jar` 并将该 `jar` 包添加进工程中。

  如果使用 `Maven` 创建工程则需要在 `pom` 文件中添加相关依赖

  ```xml
  <dependency>
  	<groupId>mysql</groupId>
  	<artifactId>mysql-connector-java</artifactId>
  	<version>5.1.47</version>
  </dependency>
  ```

- **JDBC常用接口：**

  **（1）Driver接口：**

  Driver接口由数据库厂家提供，如果要连接数据库就必须要先装载特定厂商的数据库连接驱动程序。

  **（2）Connection接口：**

  通过 `Connection` 获得一个数据库连接对象，以此对数据库进行操作。

  **（3）Statement接口：**

  用于执行SQL语句并返回生成的结果集。

  常用Statement方法： 

  **execute(String sql)：**运行语句，返回是否有结果集 

  **executeQuery(String sql)：**运行select语句，返回ResultSet结果集。 

  **executeUpdate(String sql)：**运行insert/update/delete操作，返回更新的行数。 

  **（4）ResultSet接口：**

  存储语句执行后返回的结果集。

  ResultSet提供检索不同类型字段的方法，常用的有： 

   getString(int index)、getString(String columnName)：获得在数据库里是varchar、char等类型的数据对象。 

  getFloat(int index)、getFloat(String columnName)：获得在数据库里是Float类型的数据对象。 

  getDate(int index)、getDate(String columnName)：获得在数据库里是Date类型的数据。 

  getBoolean(int index)、getBoolean(String columnName)：获得在数据库里是Boolean类型的数据。 

  getObject(int index)、getObject(String columnName)：获取在数据库里任意类型的数据。

- **使用JDBC：**

  **（1）注册驱动：**

  ```java
  Class.forName("com.mysql.jdbc.Driver");//加载驱动只需进行一次
  ```

  **（2）获得连接对象：**

  ```java
  connection = DriverManager.getConnection(url, username, password);
  //url：数据库地址    jdbc:mysql://localhost:3306/jdbc_db?useSSL=false
  //username：数据库设置的用户名
  //password：数据库密码
  ```

  **（3）发送SQL语句：**

  ```java
  String sql = "select * from book";
  result = statement.executeQuery(sql);
  ```

  **（4）处理返回结果集：**

  ```java
  while (result.next()) {
  	System.out.println(result.getInt(1) + "\t" + result.getString(2) + "\t" + 									result.getString(3) + "\t" + result.getString(4) + "\t");
  }
  ```

  **（5）释放资源：**

  ```java
  try {
  	if (resultSet != null) {
  		resultSet.close();
  	}
  } catch (SQLException e) {
  	e.printStackTrace();
  } finally {
  	try {
  		if (statement != null) {
  			statement.close();
  		}
  	} catch (SQLException e) {
  		e.printStackTrace();
  	} finally {
  		try {
  			if (connection != null) {
  				connection.close();
  			}
  		} catch (SQLException e) {
  			e.printStackTrace();
  		}
  	}
  } 
  ```

#### 3、SQL注入：

SQL注入就是将`SQL命令`或者相应的`逻辑判断`插入SQL语句中，在语句执行过程中会受这些命令或逻辑的影响，从而影响正常的程序运行结果。

例如：

```java
//根据输入的用户名和密码进行登录验证
public void login(String username, String password){
    String url = "jdbc:mysql://localhost:3306/jdbc_db?useSSL=false";
	String username = "root";
	String password = "root";
		
	Connection connection = null;
	Statement statement = null;
	ResultSet result = null;
	try {
		//1、注册驱动
		Class.forName("com.mysql.jdbc.Driver");
		//2、获得连接对象
		connection = DriverManager.getConnection(url, username, password);
		statement = connection.createStatement();
		//3、发送SQL语句
		String sql = "select * from user where name = " + username + "and psw = " + passord;
		result = statement.executeQuery(sql);
        if (result != null){
            System.out.println("登录成功");
        } else {
            System.out.println("登录失败");
        }
	} catch (ClassNotFoundException e) {
		e.printStackTrace();
	} catch (SQLException e) {
		e.printStackTrace();
	}
}
```

这样的语句在正常情况下不会有什么错误，但如果用户在输入正确的用户名后有在用户名后添加注释命令：`用户名 --` 在MySQL中可以使用 `--` 注释语句。这样一来在拼接sql语句中就回将用户名后面的语句注释掉。在只知道用户名的情况下同样能够登录成功。

**防止SQL注入：**

为了防止SQL注入问题的出现，可以使用 `PreparedStatement` 对SQL语句进行预编译，sql注入只对sql语句的准备（编译）过程有破坏作用  而`PreparedStatement`已经将sql语句准备好了，执行阶段只是把输入串作为数据处理，而不再对sql语句进行解析，准备，因此也就避免了sql注入问题。

```java
//3、发送SQL语句
String sql = "select * from user where name = ? and psw = ?";
statement = connection.prepareStatement(sql); //预编译SQL语句
statement.setString(1, username);//设置参数
statement.setString(2, password);
```

在需要填入参数的地方用问号 `?` 作为占位符，然后对SQL语句进行预编译，在将参数设置进预留的问号处，此时只会讲传入的参数作为整个内容进行执行，不会在影响SQL语句，从而避免了SQL注入问题。