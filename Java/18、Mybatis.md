## Mybatis

#### 1、什么是Mybatis：

MyBatis 是一款优秀的持久层框架，它支持定制化 SQL、存储过程以及高级映射。MyBatis 避免了几乎所有的 JDBC 代码和手动设置参数以及获取结果集。MyBatis 可以使用简单的 XML 或注解来配置和映射原生信息，将接口和 Java 的 POJOs(Plain Old Java Objects,普通的 Java对象)映射成数据库中的记录。

#### 2、ORM思想：

对象关系映射（Object Relational Mapping，简称ORM）是一种为了解决面向对象与关系型数据库之间存在互不匹配问题的技术。ORM是通过使用描述对象和数据库之间映射的元数据，将java中的对象自动持久化到关系型数据库中。

| 面向对象（Java） | 面向关系（关系型数据库） |
| :--------------: | :----------------------: |
|        类        |            表            |
|       对象       |         一行数据         |
|       属性       |          一个列          |

当实体类中的属性名和数据库表中的字段名不一致时，此时使用 JdbcTemplate 等进行数据库操作时可能会出现错误，ORM 思想就相当于在 **实体类（O）** 和 **数据库（R）** 之间添加一个中间转换介质 **（Mapping）** ，使实体类中的属性名和数据库中的字段名一一对应，这时再进行数据库的操作就不会出现字段不匹配的情况。

**ORM框架：**

1、 **JPA：** 本身是一种ORM规范，不是ORM框架

2、 **Hibernate：** 设计灵巧、性能优秀的ORM框架

3、 **Mybatis：** 最受欢迎的持久层解决方案，严格意义上Mybatis并不是一个ORM框架，是一个SQL映射框架

#### 3、准备工作：

​	1、创建数据表：

```sql
-- ----------------------------
-- Table structure for t_user
-- ----------------------------
DROP TABLE IF EXISTS `t_user`;
CREATE TABLE `t_user`  (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(20) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  `salary` decimal(8, 2) NULL DEFAULT NULL,
  PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 1 CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;
SET FOREIGN_KEY_CHECKS = 1;
```

2、创建Java项目。

#### 4、Mybatis配置文件：

##### 4.1、Mybatis全局配置文件（主配置文件）：

起名：不固定，但一般简明知义（mybatis-config.xml）

路径：classpath的根路径

参照：Mybatis官方文档

内容：1、全局的配置信息；2、属性配置信息；3、插件配置信息；4、环境配置信息；5、关联映射文件

- **配置环境：**

  MyBatis 可以配置成适应多种环境，这种机制有助于将 SQL 映射应用于多种数据库之中， 现实情况下有多种理由需要这么做。例如，开发、测试和生产环境需要有不同的配置 。

  ```xml
  <!--配置数据库的环境  default 属性用来选择默认的环境-->
  <environments default="dev">
      <!--开发环境-->
      <environment id="dev">
          <!--①事物管理器-->
          <transactionManager type="JDBC"/>
          <!--②连接池-->
          <dataSource type="POOLED">
              <!--③数据源-->
              <property name="driver" value="com.mysql.jdbc.Driver"/>
              <property name="url" value="jdbc:mysql:///mybatis_db?usrSSL=false"/>
              <property name="username" value="root"/>
              <property name="password" value="root"/>
          </dataSource>
      </environment>
  </environments>
  ```

- **关联映射文件：**

  定义SQL映射语句，需要告诉 MyBatis 到哪里去找到这些语句 ，可以使用相对于类路径的资源引用， 或完全限定资源定位符（包括 `file:///` 的 URL），或类名和包名等 

  ```xml
  <!-- 使用相对于类路径的资源引用 -->
  <mappers>
      <mapper resource="org/mybatis/builder/AuthorMapper.xml"/>
      <mapper resource="org/mybatis/builder/BlogMapper.xml"/>
      <mapper resource="org/mybatis/builder/PostMapper.xml"/>
  </mappers>
  
  <!-- 使用完全限定资源定位符（URL） -->
  <mappers>
      <mapper url="file:///var/mappers/AuthorMapper.xml"/>
      <mapper url="file:///var/mappers/BlogMapper.xml"/>
      <mapper url="file:///var/mappers/PostMapper.xml"/>
  </mappers>
  
  <!-- 使用映射器接口实现类的完全限定类名 -->
  <mappers>
      <mapper class="org.mybatis.builder.AuthorMapper"/>
      <mapper class="org.mybatis.builder.BlogMapper"/>
      <mapper class="org.mybatis.builder.PostMapper"/>
  </mappers>
  
  <!-- 将包内的映射器接口实现全部注册为映射器 -->
  <mappers>
      <package name="org.mybatis.builder"/>
  </mappers>
  ```

##### 4.2、Mybatis映射文件（Mapper文件）：

起名：不固定，但一般简明知义（xxxMapper.xml）是哪一个对象的映射文件

路径：Mapper文件应该放到Mapper接口的路径

参照：Mybatis官方文档

内容：1、编写增删改查的SQL；2、结果集映射（解决表中的列和对象中的属性不匹配的问题）；3、缓存映射

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<!--不同的mapper文件的 namespace 是不一样的-->
<mapper namespace="com.yunhui.mybatis.UserMapper">
    <!--
        select元素：专门用来做查询的SQL
        id属性：唯一标志，用来表示某一条SQL语句
        id属性和 mapper 的 namespace 唯一的表示了应用中的某一条SQL语句
        parameterType属性：表示执行该SQL语句需要的参数的类型，建议不写，MyBatis可以自行推断出来
        resultType属性：这条语句中返回的期望类型的类的完全限定名或别名,注意如果是集合情形，应该是集合可以包含的类型，而不能是集合本身，即把结果集中的每一行数据封装成什么类型的对象
    -->
    <select id="get" parameterType="java.lang.Integer" resultType="com.yunhui.mybatis.User">
        select `id`, `name`, `salary` from t_user where id = #{id}
    </select>
</mapper>
```

#### 5、日志框架：

日志框架可以把日志输出和代码分离，日志框架可以方便地定义日志的输出环境（控制台、文件、数据库），日志框架可以方便的定义日志的输出格式和输出级别。

- **常见的日志框架：** 

  | 日志门面（日志的抽象层）                                     | 日志实现                                                     |
  | :----------------------------------------------------------- | :----------------------------------------------------------- |
  | ~~JCL(Jakarta Commons Logging)~~         **SLF4J** (Simple Logging Facade for Java) | Log4J         JUL(java.util.logging)          Log4J2       **Logback** |

  一般使用：从左边选择一个接口，右边选择一个实现。

  日志门面：**SLF4J**                 			日志 实现： **Logback**  

  **Commons-logging：** （日志的规范，本身不提供实现，可以通过动态查找机制去找出真正的日志实现）

  **SLF4J：** 本质也是日志的规范

  **Log4J：** 功能强大，把日志输出到文件、控制台

  **Log4J2：** 本质是Log4J的升级，进行性能优化

- **SLF4J的使用：** 

  日志记录方法的调用，不应该直接来调用日志的实现类，而是调用日志的抽象层里的方法。

  ```java
  //首先需要导入 SLF4j 的jar包和 logback 的实现 jar
  import org.slf4j.Logger;
  import org.slf4j.LoggerFactory;
  
  public class HelloWorld {
      public static void main(String[] args) {
          //获取日志记录器 参数为类的类类型
          Logger logger = LoggerFactory.getLogger(HelloWorld.class);
          logger.info("Hello World");
      }
  }
  ```

  ![SLF4J的使用](G:\云汇科技\笔记\浙思云汇\assets/concrete-bindings.png)

- **日志的级别（从高到低）：**

  日志的级别越低输出的信息越详细。如果把日志级别设置为INFO，可以打印INFO、WARN、ERROR的信息。

  > **ERROR > WARN > INFO > DEBUG > TRACE**

  ```properties
  #log4j日志配置函数
  #设置全局的日志配置：输出DEBUG级别，输出到控制台
  log4j.rootLogger=DEBUG, stdout
  #设置自定义的日志级别
  log4j.logger.com.mybatis=TRACE
  # Console output...
  log4j.appender.stdout=org.apache.log4j.ConsoleAppender
  log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
  log4j.appender.stdout.layout.ConversionPattern=%5p [%t] - %m%n
  ```

#### 6、抽取MybatisUtil

- **SqlSessionFactoryBuilder**

  这个类可以被实例化、使用和丢弃，一旦创建了 SqlSessionFactory，就不再需要它了。因此 SqlSessionFactoryBuilder 实例的最佳作用域是方法作用域（也就是局部方法变量）。可以重用 SqlSessionFactoryBuilder 来创建多个 SqlSessionFactory 实例，但是最好还是不要让其一直存在以保证所有的 XML 解析资源开放给更重要的事情。

- **SqlSessionFactory**

  SqlSessionFactory 一旦被创建就应该在应用的运行期间一直存在，没有任何理由对它进行清除或重建。使用 SqlSessionFactory 的最佳实践是在应用运行期间不要重复创建多次，多次重建 SqlSessionFactory 被视为一种代码“坏味道（bad smell）”。因此 SqlSessionFactory 的最佳作用域是应用作用域。有很多方法可以做到，最简单的就是使用单例模式或者静态单例模式。

- **SqlSession**

  每个线程都应该有它自己的 SqlSession 实例。SqlSession 的实例不是线程安全的，因此是不能被共享的，所以它的最佳的作用域是请求或方法作用域。绝对不能将 SqlSession 实例的引用放在一个类的静态域，甚至一个类的实例变量也不行。

```java
public class MybatisUtil {
    private MybatisUtil(){}
    private static SqlSessionFactory factory = null;
    static {
        try {
            //创建SqlSessionFactory对象
            factory = new SqlSessionFactoryBuilder().build(Resources.getResourceAsStream("mybatis-config.xml"));
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    //返回一个SqlSession对象
    public static SqlSession getSqlSession(){
        return factory.openSession();
    }
}
```

#### 7、Mybatis的CRUD操作：

- ##### Select:

  ```xml
  <!--Mapper映射文件-->
  <select id="get" parameterType="java.lang.Integer" resultType="com.mybatis.User">
      select `id`, `name`, `salary` from t_user where id = #{id}
  </select>
    
  <select id="getAll" resultType="com.mybatis.User">
      select `id`, `name`, `salary` from t_user
  </select>
  ```

  ```java
  /**
   * 查询用户id为1的用户信息
   */
  @Test
  public void test1() throws IOException {
      //加载Mybatis全局配置文件并创建SqlSessionFactory对象
      SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(Resources.getResourceAsStream("mybatis-config.xml"));
      //创建一个SqlSession对象
      SqlSession session = factory.openSession(false);//
      User u = session.selectOne("com.mybatis.UserMapper.get",1);
      System.out.println(u);
      //关闭SqlSession
      session.close();
  }
  ```

  ```java
  /**
   * 查询所有的用户信息
   * @throws IOException
   */
  @Test
  public void test2() throws IOException {
      //加载Mybatis全局配置文件并创建SqlSessionFactory对象
      SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(Resources.getResourceAsStream("mybatis-config.xml"));
      //创建一个SqlSession对象
      SqlSession session = factory.openSession(false);
      List<User> users = session.selectList("com.mybatis.UserMapper.getAll");
      for (User user : users) {
          System.out.println(user);
      }
      //关闭SqlSession
      session.close();
  }
  ```

  > openSession(true)：当设置为 true 时session会话会自动提交事务，否则需要手动设置 session.commit() 只在插入、更新、删除时有效。

-  ##### update：

   ```xml
   <update id="updateUser" parameterType="com.mybatis.User">
       update t_user set `name` = #{name} where id = #{id}
   </update>
   ```

-  ##### insert：

   ```xml
   <insert id="insterUser" parameterType="com.mybatis.User">
       insert into t_user values(null, #{name}, #{salary})
   </insert>
   ```

   **useGeneratedKeys：** （仅对 insert 和 update 有用）这会令 MyBatis 使用 JDBC 的 getGeneratedKeys 方法来取出由数据库内部生成的主键（比如：像 MySQL 和 SQL Server 这样的关系数据库管理系统的自动递增字段），默认值：false。 

   **keyProperty：** （仅对 insert 和 update 有用）唯一标记一个属性，MyBatis 会通过 getGeneratedKeys 的返回值或者通过 insert 语句的 selectKey 子元素设置它的键值，默认：`unset`。如果希望得到多个生成的列，也可以是逗号分隔的属性名称列表。 

-  ##### delete：

   ```xml
   <delete id="deleteUser" parameterType="java.lang.Integer">
       delete from t_user where id = #{id}
   </delete>
   ```

#### 8、typeAliases类型别名：

类型别名是为 Java 类型设置一个短的名字。它只和 XML 配置有关，存在的意义仅在于用来减少类完全限定名的冗余。 在全局配置文件中设置。 **Mybatis中别名是不区分大小写的。** 

```xml
<!--类型别名-->
<typeAliases>
    <!--
        type：类型的全路径名称
        alias：该类型的简单名称
    -->
    <typeAlias type="com.mybatis.User" alias="user"/>
    <!--
        name：类所在的包名，配置后会自动为包中的类起别名，默认是简单类名首字母小写
    -->
    <package name="com.mybatis"/>
</typeAliases>
```

**注意：** 可以使用 @Alias("别名") 的注解为某一个类单独起别名

#### 9、使用Mapper接口：

Mapper 组件：Mapper 接口 + Mapper文件。

1、Mapper 文件和 Mapper 接口应该放在同一个包中；

2、Mapper 文件中的 namespace 就设置为对应 Mapper 接口的全限定名称；

3、Mapper 文件中操作元素的 id 与 Mapper 接口中的方法名称一一对应。

**Mapper 接口的原理：** 动态代理，代理对象 class com.sun.proxy.$Proxy5 

- **Mybatis的参数处理：**

  方式一：封装POJO（Java对象），但需要定义很多的 JavaBean。

  方式二：使用 Map 封装多个参数。

  方式三：使用 @Param 注解，其底层原理就是方式二，@Param(“”) 注解中的字符串相当于Map中的key。

- **Mybatis中的#和$：**

  1、都可以通过#和$来获得对象中的信息。

  2、使用 # 传递的参数会先转换为占位符 ? ，再通过设置占位符参数的方式来设置值；使用 $ 传递的参数直接会把解析出来的数据作为SQL语句的一部分。

  **如何选择：**

  1、如果需要设置占位符参数的地方，全部使用 #

  2、如果写的内容需要作为SQL的一部分，此时应该使用 $

#### 10、使用注解进行开发：

```java
@Select("select `id`, `name`, `salary` from t_user where id = #{id}")
User getUser(int id);

@Insert("insert into t_user (name, salary) values(#{name}, #{salary})")
@Options(useGeneratedKeys=true, keyProperty="id")
void save(User user);
```

```java
@Select("select `id`, `name`, `salary` from t_user where id = #{id}")
User getUser(int id);

@Select("select `id`, `name`, `salary` from t_user")
List<User> getAllUser();

//配置结果集映射
@Results(id = "resultMap", value = {
        @Result(column = "", property = ""),
        @Result(column = "", property = ""),
        @Result(column = "", property = "")
})

@ResultMap("resultMap")//使用指定的结果集映射
```

#### 11、动态SQL：

- ##### if：

  在 test 中写判断条件。

  ```xml
  <select id="findActiveBlogWithTitleLike" resultType="Blog">
      SELECT * FROM BLOG 
      WHERE state = ‘ACTIVE’ 
      <if test="title != null"> <!--如果条件成立则执行 if 标签内的语句进行SQL拼接-->
          AND title like #{title}
      </if>
  </select>
  ```

- ##### choose, when, otherwise：

  ```xml
  <select id="findActiveBlogLike" resultType="Blog">
      SELECT * FROM BLOG WHERE state = ‘ACTIVE’
      <choose>
          <when test="title != null">
              AND title like #{title}
          </when>
          <when test="author != null and author.name != null">
              AND author_name like #{author.name}
          </when>
          <otherwise>
              AND featured = 1
          </otherwise>
        </choose>
  </select>
  ```

- ##### where：

  当使用动态SQL时会出现 where 关键字后面没有条件的情况，这样会造成SQL错误，虽然可以通过 where 1=1来解决这问题，但这显然不是最好的解决方法。这时候可以使用 \<where> 标签来解决。

  where 元素会判断查询条件是否有 where 关键字，如果有则在第一个条件前插入一个 where 关键字，如果查询条件以 and 或者 or 开头会把第一个条件的 and 或 or 替换成 where。

  ```xml
  <select id="findActiveBlogLike" resultType="Blog">
      SELECT * FROM BLOG 
      <where> 
          <if test="state != null">
              state = #{state}
          </if> 
          <if test="title != null">
              AND title like #{title}
          </if>
          <if test="author != null and author.name != null">
              AND author_name like #{author.name}
          </if>
      </where>
  </select>
  ```

- #### set：

  根据 set 中的 sql 语句动态地去掉最后一个逗号，并在最前面加上一个 set 关键字。如果 set 元素没有内容会自动忽略 set 语句。

  ```xml
  <update id="updateAuthorIfNecessary">
      update Author
          <set>
              <if test="username != null">username=#{username},</if>
              <if test="password != null">password=#{password},</if>
              <if test="email != null">email=#{email},</if>
              <if test="bio != null">bio=#{bio}</if>
          </set>
      where id=#{id}
  </update>
  ```

- ##### trim:

  前提： 如果trim元素包含内容返回一个字符串

  **prefix：** 在这个字符串之前插入 prefix 属性值；

  **prefixOverrides：** 字符串中的内容以 prefixOverrides 中的内容开头，则用 prefix 中的内容取代prefixOverrides  指定的开头内容。

  **suffix：** 在这个字符串之后插入 suffix 属性值；

  **suffixOverrides：** 字符串中的内容以 suffixOverrides 中的内容结束，则用 suffix 中的内容取代suffixOverrides 指定的结尾内容。

  ```xml
  <trim prefix="" prefixOverrides="" suffix="" suffixOverrides="">
              
  </trim>
  ```

- ##### foreach：

  对一个集合进行遍历，通常是在构建 IN 条件语句的时候使用。

  **collection：** 表示对哪一个集合或数组进行迭代，如果参数是 **数组** （Mybatis会把数组、集合封装成 map），此时Map的key为 array，如果参数是 **List** 类型，此时 map 的 key 为 list。

  **item：** 表示被迭代的每一个元素的变量。

  **index：** 迭代的索引。

  **open：** 在迭代集合前拼接什么符号。

  **close：** 在迭代集合之后拼接什么符号。

  **separator：** 在迭代元素时，元素之间使用说明符号分隔。

  ```xml
  <select id="selectPostIn" resultType="domain.blog.Post">
      SELECT *
      FROM POST P
      WHERE ID in
      <foreach collection="list" item="item" index="index"  open="(" separator="," close=")">
          #{item}
      </foreach>
  </select>
  ```

#### 12、高级查询：

```java
/**
 * 封装用户高级查询的信息
 */
@Data
public class UserQueryObject {

    private String keyWord;//关键字
    private BigDecimal minSalary;//最低工资
    private BigDecimal maxSalary;//最高工资
    private Integer id = -1;//用户id，缺省为-1表示为所有用户

    //如果字符串为空串，也返回 null
    public String getKeyWord(){
        return empty2null(this.keyWord);
    }

    private String empty2null(String str){
        return hasLength(str)? str : null;
    }

    private boolean hasLength(String str){
        return str != null && !"".equals(str.trim());
    }
}
```

```java
//接口方法
List<User> queryForList(UserQueryObject userQueryObject);

int queryForCount(UserQueryObject userQueryObject);
```

```xml
<!--Mapper映射文件-->
<sql id="base_where">
    <where>
        <if test="keyWord != null">
            and `name` like concat('%',#{keyWord},'%')
        </if>
        <if test="minSalary != null">
            and `salary` >= #{minSalary}
        </if>
        <if test="maxSalary != null">
            and `salary` &lt;= #{maxSalary}
        </if>
    </where>
</sql>
    
<select id="queryForList" resultType="User">
    select `id`, `name`, `salary` from t_user
    <include refid="base_where"/>
</select>

<select id="queryForCount" resultType="int">
    select count(`id`) from t_user
    <include refid="base_where"/>
</select>
```

#### 13、对象关系：

在Mybatis中查询的数据需要使用一个对象进行封装，通常情况下一个对象的功能是有限的，如果需要完成复杂的功能就需要对象之间的协作，也就产生了对象之间的关系。例如：泛化关系、实现关系、依赖关系、关联关系、聚合关系、组合关系……

- ##### 泛化关系：

  实际就是 **继承关系** 比如类和类之间、接口和接口之间，使用 extends 表示。在 UML 中，继承通常是使用空心三角+实线来表示。

- ##### 实现关系：

  其实就是实现关系，存在类和接口之间，使用implements表示，在UML中实现通常是空心三角+虚线表示。

- ##### 依赖关系：

  表示一个A类依赖另一个B类的定义，如果A对象离开了B对象，A就不能正常编译。在UML中依赖通常使用虚线箭头表示。

- ##### 泛化关系的表设计：

  ```java
  //普通用户
  public class User{
      private Long id;
      private String name;
  }
  
  //员工
  public class Employee extends User{
      private BigDecimal salary;
  }
  
  //客户
  public class Customer extends User{
      private String address;
  }
  ```

  类之间的关系：员工类继承普通用户类，客户继承普通用户类。

  在面向对象的继承关系中设计表时有三种情况：（单表查询效率是最高的）

  1、共用一张表：一张表中同时存储用户、员工、客户的信息；可以添加一个类型字段用来区分每条数据的性质，是用户、员工还是客户。缺点：不能添加字段约束

  2、每个子类一张表：将共有的信息作为一个表，第一个表有id、name字段，第二个表有id、salary字段，第三个表有id、address字段。后面两个表通过外键关联第一个表。

  3、每个类一张表：数据有冗余，id可能出现错乱。

- ##### 关联关系：

  A 对象依赖 B 对象，并且把 B 作为 A 的一个成员变量，则 A 和 B 存在关联关系，在UML中依赖通常使用实线箭头表示。

  - **按照导航性分：** 如果通过 A 对象中的某一个属性可以访问到 B 对象，则说 A 可以导航到 B。

    单向：只能从 A 通过属性导航到 B ，B 不能导航到 A。

    双向：A 可以通过属性导航到 B ， B 也可以通过属性导航到 A。

  - **按照多重性分：** 

    **一对一：** 一个 A 对象属于一个 B 对象，一个 B 对象属于一个 A 对象。一个QQ号码对应一个QQ空间

    ```java
    //QQ号码
    public class QQNumber{
        private QQZone zone;
    }
    //QQ空间
    public class QQZone{
        
    }
    //通过QQ号码可以找到QQ空间，单向一对一
    
    //QQ号码
    public class QQNumber{
        private QQZone zone;
    }
    //QQ空间
    public class QQZone{
        private QQNumber number;
    }
    //通过QQ号码可以找到QQ空间，也可以通过QQ空间找到QQ号码，双向一对一
    ```

    **一对一表的设计：** 方式一，添加唯一外键约束，例如在QQ空间表中添加外键关联QQ的ID，并对该列做唯一约束；方式二，共享主键，QQ号码表的主键和QQ空间的主键一致。在开发中更多使用方式一，通过业务代码做到唯一。

    **一对多：** 一个 A 对象包含多个 B 对象。

    ```java
    public class Department{
        private List<Employee> emps = new ArrayList<>();//一个部门对应多个员工
    }
    
    public class Employee{
        
    }
    ```

    **多对一：** 多个 A 对象属于一个 B 对象，并且每个 A 对象只能属于一个 B 对象。

    ```java
    public class Department{
    }
    
    public class Employee{
        private Department dept;//多个员工对应同一个部门
    }
    ```

    **多对多：** 一个 A 对象属于多个 B 对象，一个 B 对象属于多个 A 对象。

    ```java
    public class Student{
        private List<Teacher> teachers = new ArrayList<>();
    }
    
    public class Teacher{
        
    }
    //一个老师有多个学生，一个学生也可以有多个老师。单向多对多
    ```

#### 14、对象关系映射：

```xml
<resultMap id="stuCardMap" type="StuCard">
    <id property="cardId" column="cardid"/>
    <result property="loginName" column="loginname"/>
    <result property="loginPwd" column="loginpwd"/>
    <result property="createDate" column="createdte"/>
    <association property="student" javaType="Student" column="sid" select="com.yunhui.mybatis.dao.StudentDaoMapper.getStudentById"/>
</resultMap>
```

对象 A 中包含对象 B 时SQL的写法：select 需要查询的语句（命名空间+id值）， column 传递给目标查询语句的值，

**N + 1 问题：** 假如每个员工的部门 ID 是不同，查询所有的员工信息就会发生 N + 1 问题。其实就是发送了 N + 1 条SQL语句。

1： select * from employee

N： select * from department where id = ?

导致 N + 1 问题的原因在于使用了额外的 SQL 语句去查询。

```xml
<resultMap id="resultMap" type="Employee">
    <id property="id" column="id"/>
    <result property="name" column="name"/>
    <association property="dept" column="dept_id" select="com.mybatis.mapper.DepartmentMapper.get"/>
</resultMap>

<select id="getList" resultType="com.mybatis.entity.Employee">
    select `id`, `name` from employee
</select>
```

解决方案： 使用多表联查，一条SQL语句搞定。使用内联映射处理多表联查的结果。

```xml
<resultMap id="resultMap" type="Employee">
    <id property="id" column="id"/>
    <result property="name" column="name"/>
    <association property="dept" javaType="Department">
        <id property="did" column="did"/>
        <result property="dname" column="dname"/>
    </association>
</resultMap>

<select id="getList" resultType="com.mybatis.entity.Employee">
    select e.`id`, e.`name`, d.`did`, d.`dname` from employee e join department d on e.dept_id = d.did
</select>
```

**一对多内联查询：** 

```xml
<resultMap id="" type="">
	<id property="" column=""/>
    <result property="" column=""/>
    <collection property="" ofType="">
    	<id property="" column=""/>
    	<result property="" column=""/>
    </collection>
</resultMap>
```

**ofType：** 表示集合中的泛型。

#### 15、延迟加载：

为了避免一些性能开销而提出的一个概念，在真正需要使用到数据的时候，才去执行SQL语句去查询。

**lazyLoadingEnabled：** 延迟加载的全局开关。当开启时，所有关联对象都会延迟加载。 特定关联关系中可通过设置`fetchType`属性来覆盖该项的开关状态。

**lazyLoadTriggerMethods ：** 指定哪个对象的方法触发一次延迟加载。 

```xml
<settings>
    <!--开启延时加载-->
    <setting name="lazyLoadingEnabled" value="true"/>
    <!--设置不积极地去查询关联对象-->
    <setting name="aggressiveLazyLoading" value="false"/>
    <!--指定哪个对象的方法触发一次延迟加载-->
    <setting name="lazyLoadTriggerMethods" value="clone"/>
</settings>
```

> **注意点：**
>
> 1、针对单属性对象，使用 association 元素，通常直接使用多表操作，也就是使用内联查询。
>
> 2、针对集合对象，使用 collection 元素，通常使用延时加载，也就是额外SQL处理。

#### 16、Mybatis的缓存机制：

缓存（Cache）：可以使应用更快地获取数据，避免和数据库做频繁的交互操作，尤其在查询操作较多的时候。

原理：Map，查询的时候先从缓存中去查询数据，如果找到数据，立即返回；如果找不到，去查询数据库，并放进缓存中，以供下次使用，然后返回数据。

**Mybatis的缓存：** 

- **一级缓存（本地缓存）：** 默认开启，不能关闭

  在 SqlSession 中存在一个 Map 用来缓存查询出来的对象。一级缓存对性能的提升是有限的。多个SqlSession之间是不共享一级缓存的。真正的提升性能需要使用二级缓存。

  SqlSession 级别的，每次创建新的 SqlSession 对象，一级缓存空间就改变了，不同的 SqlSession 对象不共享数据。

- **二级缓存（查询缓存）：** 需要手动开启和配置，也可以使用第三方缓存技术  EhCache。

  > 1、哪些数据适合放到缓存中？
  >
  > ​		一般情况下，经常被查询且很少被修改的数据，读远远大于写的操作数据。
  >
  > 2、缓存性能相关的属性：
  >
  > **命中率：** 从缓存中查询出来的数量/总的查询数量：查询1000条数据，其中800条来之缓存，800/1000=80%
  >
  > **最大对象数量：** 缓存区域中最多存储多少个数据，数据较多可以写入到本地磁盘中（序列化）。
  >
  > **最大空闲时间：** 数据在缓存区域存在的最大时间。

  mapper 级别的，二级缓存的作用域是 mapper 文件的同一个 namespace。 二级缓存应该要和 namespace 绑定在一起，不同的 SqlSession 对象是共享数据的。

  在 Mybatis 中实现缓存，只需要实现 Cache 接口即可，Mybatis 已经提供了一个自带的缓存技术。

  **使用二级缓存：** 

  1、在全局配置文件中启用二级缓存：

  ```xml
  <settings>
      <!--启用二级缓存，默认已经启用-->
      <setting name="cacheEnabled" value="true"/>
  </settings>
  ```

  2、在 mapper 文件中，使用 cache 元素把 namespace 和缓存绑定：

  ```xml
  <mapper namespace="">
  	<cache
    		eviction="LRU"
    		flushInterval="60000"
    		size="512"
    		readOnly="true"/>
  </mapper>
  ```

  **cache属性：**

  **eviction：** 缓存的回收策略：

  `LRU` – 最近最少使用的:移除最长时间不被使用的对象。

  `FIFO` – 先进先出:按对象进入缓存的顺序来移除它们。

  `SOFT` – 软引用:移除基于垃圾回收器状态和软引用规则的对象。

  `WEAK` – 弱引用:更积极地移除基于垃圾收集器状态和弱引用规则的对象。

  **flushInterval(刷新间隔)：** 可以被设置为任意的正整数,而且它们代表一个合理的毫秒 形式的时间段 。默认情况是不设置,也就是没有刷新间隔,缓存仅仅调用语句时刷新。

  **size(引用数目)：** 可以被设置为任意正整数,要记住你缓存的对象数目和你运行环境的 可用内存资源数目。默认值是 1024。 

  **readOnly(只读)：** 属性可以被设置为 true 或 false。只读的缓存会给所有调用者返回缓 存对象的相同实例。 默认是 false 

  3、把放入二级缓存的对象实现序列化接口：

  ```java
  public class EmployeeExample implements Serializable {
  
  }
  ```

二级缓存的配置细节：

当启用二级缓存后，mapper 文件中所有的 select 元素都会使用缓存；在大多数情况下，针对列表查询（查询多条数据），设置为不使用缓存，只有 SQL 和参数相同的时候才会用到缓存；一般只会对 get 方法做查询缓存；默认情况下 insert、update、delete 操作都会去刷新缓存。

- **Ehcache：** 配置文件

  ```xml
  <ehcache xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:noNamespaceSchemaLocation="http://code.taobao.org/p/tgw/diff/3/tgw/src/ehcache.xsd">
  
  
      <!--<diskStore path="java.io.tmpdir"/>-->
      <diskStore path="C:\ehcache"/>
  
      <!--   
         name:缓存名称。   
         maxElementsInMemory：缓存最大个数。   
         eternal:对象是否永久有效，一但设置了，timeout将不起作用。   
         timeToIdleSeconds：设置对象在失效前的允许闲置时间（单位：秒）。仅当eternal=false对象不是永久有效时使用，可选属性，默认值是0，也就是可闲置时间无穷大。   
         timeToLiveSeconds：设置对象在失效前允许存活时间（单位：秒）。最大时间介于创建时间和失效时间之间。仅当eternal=false对象不是永久有效时使用，默认是0.，也就是对象存活时间无穷大。   
         overflowToDisk：当内存中对象数量达到maxElementsInMemory时，Ehcache将会对象写到磁盘中。   
         diskSpoolBufferSizeMB：这个参数设置DiskStore（磁盘缓存）的缓存区大小。默认是30MB。每个Cache都应该有自己的一个缓冲区。   
         maxElementsOnDisk：硬盘最大缓存个数。   
         diskPersistent：是否缓存虚拟机重启期数据 Whether the disk store persists between restarts of the Virtual Machine. The default value is false.   
         diskExpiryThreadIntervalSeconds：磁盘失效线程运行时间间隔，默认是120秒。   
         memoryStoreEvictionPolicy：当达到maxElementsInMemory限制时，Ehcache将会根据指定的策略去清理内存。默认策略是LRU（最近最少使用）。你可以设置为FIFO（先进先出）或是LFU（较少使用）。   
         clearOnFlush：内存数量最大时是否清除。   
      -->
      
      <defaultCache
              maxElementsInMemory="10000"
              eternal="false"
              timeToIdleSeconds="120"
              timeToLiveSeconds="120"
              overflowToDisk="true"
              maxElementsOnDisk="10000000"
              diskPersistent="false"
              diskExpiryThreadIntervalSeconds="120"
              memoryStoreEvictionPolicy="LRU"
      />
  	<!--自定义区域的cache  name=mapper文件的namespace-->
      <cache name="testCache"
             maxBytesLocalHeap="50m"
             timeToLiveSeconds="100">
      </cache>
  
      <!--
         maxElementsInMemory设置成1，overflowToDisk设置成true，只要有一个缓存元素，就直接存到硬盘上去
         eternal设置成true，代表对象永久有效
         maxElementsOnDisk设置成0 表示硬盘中最大缓存对象数无限大
         diskPersistent设置成true表示缓存虚拟机重启期数据
      -->
      <cache
              name="disk"
              maxElementsInMemory="1"
              eternal="true"
              overflowToDisk="true"
              maxElementsOnDisk="0"
              diskPersistent="true"/>
  </ehcache>
  ```

  使用 ehcache ：

  ```xml
  <!--使用EhCache技术-->
  <cache typr="org.mybatis.caches.ehcache.EhcacheCache"/>
  ```

#### 17、Mybatis Generator：

1、使用需要提供一个 generatorConfig.xml 配置文件，文件中包含了生成代码和配置的参数。

2、运行配置文件：方式一，使用 java 代码运行（可直接去官网拷贝）；方式二：使用 maven 插件运行。

**QBC风格：** Query By Criteria，一种查询方式。主要由 Criteria ，Example 组成，可以使用面向对象的方式去拼写查询条件，一般适用于简单查询。

#### 18、pagehelper分页插件：

https://pagehelper.github.io/docs/howtouse/

1、在Mybatis配置文件中配置分页插件：

```xml
<plugins>
    <!-- com.github.pagehelper为PageHelper类所在包名 -->
    <plugin interceptor="com.github.pagehelper.PageInterceptor">
        <!-- 使用下面的方式配置参数，后面会有所有的参数介绍 -->
        <property name="param1" value="value1"/>
	</plugin>
</plugins>
```

2、分页参数：

`helperDialect`：分页插件会自动检测当前的数据库链接，自动选择合适的分页方式。也可以手动指定。

`offsetAsPageNum`：默认值为 `false`，该参数对使用 `RowBounds` 作为分页参数时有效。 当该参数设置为 `true` 时，会将 `RowBounds` 中的 `offset` 参数当成 `pageNum` 使用，可以用页码和页面大小两个参数进行分页。

`rowBoundsWithCount`：默认值为`false`，该参数对使用 `RowBounds` 作为分页参数时有效。 当该参数设置为`true`时，使用 `RowBounds` 分页会进行 count 查询。

`pageSizeZero`：默认值为 `false`，当该参数设置为 `true` 时，如果 `pageSize=0` 或者 `RowBounds.limit = 0` 就会查询出全部的结果（相当于没有执行分页查询，但是返回结果仍然是 `Page` 类型）。

`reasonable`：分页合理化参数，默认值为`false`。当该参数设置为 `true` 时，`pageNum<=0` 时会查询第一页， `pageNum>pages`（超过总数时），会查询最后一页。默认`false` 时，直接根据参数进行查询。

`params`：为了支持`startPage(Object params)`方法，增加了该参数来配置参数映射，用于从对象中根据属性名取值， 可以配置 `pageNum,pageSize,count,pageSizeZero,reasonable`，不配置映射的用默认值， 默认值为`pageNum=pageNum;pageSize=pageSize;count=countSql;reasonable=reasonable;pageSizeZero=pageSizeZero`。

`supportMethodsArguments`：支持通过 Mapper 接口参数来传递分页参数，默认值`false`，分页插件会从查询方法的参数值中，自动根据上面 `params` 配置的字段中取值，查找到合适的值时就会自动分页。 使用方法可以参考测试代码中的 `com.github.pagehelper.test.basic` 包下的 `ArgumentsMapTest` 和 `ArgumentsObjTest`。

`autoRuntimeDialect`：默认值为 `false`。设置为 `true` 时，允许在运行时根据多数据源自动识别对应方言的分页 （不支持自动选择`sqlserver2012`，只能使用`sqlserver`），用法和注意事项参考下面的**场景五**。

`closeConn`：默认值为 `true`。当使用运行时动态数据源或没有设置 `helperDialect` 属性自动获取数据库类型时，会自动获取一个数据库连接， 通过该属性来设置是否关闭获取的这个连接，默认`true`关闭，设置为 `false` 后，不会关闭获取的连接，这个参数的设置要根据自己选择的数据源来决定。

3、使用教程：

```java
//Mapper接口方式的调用，推荐这种使用方式。
PageHelper.startPage(1, 10);
List<Country> list = countryMapper.selectIf(1);
```

