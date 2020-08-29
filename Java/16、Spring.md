## Spring

#### 1、IOC & DI 概述： [如何理解Spring IOC和DI](https://www.jianshu.com/p/dbcde99c5a98)

**IOC：** 反转控制（将原本在程序中手动创建对象的控制权交给Spring来管理），反转资源的获取方向，传统的资源获取要求组件向容器发起请求查找资源。应用IOC之后是容器主动将资源推送给它所管理的组件，组件所要做的仅是选择一种合适的方式来接受资源。这种行为也被称为查找的被动形式。控制反转可以理解为对象的赋值。

**DI：** IOC的另一种表述方式，组件以一些预先定义好的方式（例如：setter方法）接受来自容器的资源注入。DI是对象中属性的赋值。Spring在创建对象的过程中，将对象依赖的属性通过配置设置给该对象。

#### 2、Spring Bean 配置（基于XML文件的方式）：

- **（1）在IOC容器中配置Bean**

```xml
<!--
    配置Bean
    class：bean的全类名，通过反射的方式在IOC容器中创建Bean，所以要求Bean中必须存在无参的构造方法
    id：标识容器中的Bean，id是唯一的
-->
<bean id="person" class="com.spring1.Person">
    <property name="name" value="张三"/>
    <property name="age" value="20"/>
</bean>
```

**注意：** Bean中要有无参的构造方法。

在java中获取Bean

```java
@Test
public void test1(){
    //1、创建IOC容器
    /*
        ApplicationContext：代表IOC容器，实际上是一个接口
        ClassPathXmlApplicationContext：从类路径下加载配置文件
        FileSystemXmlApplicationContext：从文件系统中加载配置文件
        ApplicationContext：在初始化上下文的时候会实现所有单例的Bean
     */
    ApplicationContext ac = new ClassPathXmlApplicationContext("applicationContext.xml");
    //2、获取对象
    Person p = (Person) application.getBean("person");
    //3、调用方法
    p.message();
}
```

在 Spring IOC 容器读取Bean配置创建Bean实例之前必须对它进行初始化后，才能从容器中获取Bean实例并使用。ApplicationContext的主要实现类 ClassPathXmlApplicationContext 从类路径加载配置文件，FileSystemXmlApplicationContext 从文件系统加载配置文件。

> ApplicationContext 在初始化上下文对象时就实例化所有单例的Bean

- **（2）XML中属性注入：**

```xml
<!-- 通过setter方法注入属性 -->
<bean id="person" class="com.spring1.Person">
    <property name="name" value="张三"/>
    <property name="age" value="20"/>
</bean>
<!-- <property name="属性名" value="属性值"></property> -->
```

```xml
<!-- 
	通过构造函数创建对象
    属性值除了能通过 value 属性进行配置，还可以通过 <value></value> 标签进行配置
-->
<bean id="person2" class="com.spring1.Person">
    <constructor-arg type="java.lang.String">
        <!--如果参数里有特殊字符可以使用 <![CDATA[]]> 标签包裹起来-->
        <value><![CDATA[<李四^>]]></value>
    </constructor-arg>
    <constructor-arg type="int" value="25"/>
</bean>
```

“\<constructor-arg  value="25"/\>” 配置 \<constructor-arg\> 中值写value属性会按照Bean中的构造方法中的参数顺序进行赋值，如果有多个带参构造函数可能会出现错误。“\<constructor-arg type="int" value="25"/\>” 解决多个构造方法的可能出错的问题可以在 \<constructor-arg\> 中配置type属性，根据参数的类型进行赋值，从而选择不同的构造函数。

**注意：** 如果属性值中有特殊字符需要使用 **\<![CDATA[值]]>** 将值包裹起来。

```xml
<!-- 引用其他Bean -->
<bean id="person3" class="com.spring1.Person">
    <property name="name" value="小明"/>	   <!--配置姓名-->
    <property name="age" value="18"/>		<!--配置年龄-->
    <property name="clazz" ref="clazz"/>	<!--配置班级-->
</bean>
<!-- 在property标签中使用ref属性引用其他Bean，引用的Bean需要在外部声明 -->
```

```xml
<!-- 使用内部Bean进行属性赋值 -->
<bean id="person3" class="com.spring1.Person">
    <property name="name" value="小明"/>
    <property name="age" value="18"/>
    <property name="clazz">
        <!--内部Bean  不能被外部Bean引用，只能在内部使用-->
        <bean class="com.yunhui.spring1.Clazz">
            <property name="clazzNum" value="112"/>
            <property name="clazzName" value="六年级"/>
        </bean>
    </property>
</bean>
```

- **（3）null 值和级联属性：**

1、可以使用专用的 \<null/> 元素标签为Bean的字符串或其它对象类型的属性注入 null 值。

2、Spring支持级联属性的配置。

```xml
<bean id="person3" class="com.spring1.Person">
    <property name="name" value="小明"/>	   <!--配置姓名-->
    <property name="age" value="18"/>		<!--配置年龄-->
    <!-- 测试赋值null -->
    <property name="clazz"><null/></property>	<!--配置班级-->
</bean>
```

```xml
<!--级联属性赋值-->
<bean id="clazz" class="com.spring1.Clazz"></bean>
<bean id="person3" class="com.yunhui.spring1.Person">
    <property name="name" value="小明"/>	   <!--配置姓名-->
    <property name="age" value="18"/>		<!--配置年龄-->
    <property name="clazz" ref="clazz"/>	<!--配置班级-->
    <property name="clazz.clazzNum" value="456"/> <!--配置班级的编号-->
    <property name="clazz.clazzName" value="高一"/> <!--配置班级的名称-->
</bean>
<!--
	注意：属性需要先初始化后才可以为级联属性赋值，否则会出现异常
-->
```

- **（4）集合属性：**

```xml
<!-- 数组 -->
<property name="">
	<array>a</array>
    <array>b</array>
    <array>c</array>
</property>

<!-- Lidst集合 -->
<property name="">
	<list>1</list>
    <list>2</list>
    <list>3</list>
</property>

<!-- Map集合 -->
<property name="">
	<map>
        <entry key="" value=""></entry>
        <entry key="" value-ref=""></entry>
    </map>
</property>

<!-- Properties -->
<property name="">
	<props>
    	<prop key="user">root</prop>
        <prop key="password">123456</prop>
        <prop key="url">jdbc:mysql:///test</prop>
        <prop key="driverClass">com.mysql.jdbc.Driver</prop>
    </props>
</property>
```

Spring中可以通过 \<list> 、 \<set>、 \<map>来配置集合属性。

- **（5）使用 utility scheme 定义集合：**

使用基本的集合标签定义集合时，不能将集合作为独立的Bean定义，导致其它Bean无法直接引用该集合，集合不能在不同的Bean之间共享。

```xml
<!-- 配置独立的集合bean，以供多个bean进行引用 -->
<!--需要导入util命名空间-->
<util:list id="cla">
    <ref bean="clazz"/>
    <ref bean="clazz2"/>
</util:list>
```

- **（6）使用p命名空间：**

为了简化XML文件的配置，Spring引入了一个新的p命名空间，可以通过 \<bean> 元素属性的方式配置 Bean 的属性。使用 p 命名空间后，基于XML方式的配置将进一步简化。

```xml
<!--
	通过 p 命名空间为 bean 的属性赋值
	使用前需要先引入 p 的命名空间 xmlns:p="http://www.springframework.org/schema/p"
-->
<bean id="person5" class="com.spring1.Person" p:age="26" p:name="Tom" p:clazz-ref="clazz"/>
```

- **（7）XML配置里的Bean自动装配：**

Spring IOC 容器可以自动装配Bean，只需要在 \<bean> 的 **autowrie 属性里指定自动装配的模式** 。

**byType：** （根据类型自动装配）若IOC容器中有多个与目标Bean类型一致的Bean，Spring将不能执行自动装配。

**byName：** （根据名称自动装配）必须将目标Bean的名称和属性名设置完全相同。

**constructor：** （通过构造器自动装配）当Bean中存在多个构造器时此装配方式将会很复杂，不推荐使用。

```xml
<!-- 手动装配 -->
<bean id="address" class="com.autowire.Address"
      p:city="Beijing" p:street="HuiLongGuan"/>

<bean id="car" class="com.autowire.Car"
      p:brand="Audi" p:price="300000"/>

<bean id="people" class="com.autowire.People"
      p:name="Tom" p:address-ref="address" p:car-ref="car"/>
```

```xml
<!-- 
	通过名称自动装配 根据Bean的名字和当前 bean 的 setter 风格的属性名进行自动装配
	若有匹配的则自动装配，若没有匹配的则不装配
-->
<bean id="address" class="com.autowire.Address"
      p:city="Beijing" p:street="HuiLongGuan"/>

<bean id="car" class="com.autowire.Car"
      p:brand="Audi" p:price="300000"/>

<bean id="people" class="com.autowire.People"
      p:name="Tom" autowire="byName"/>
```

```xml
<!-- 
	通过类型自动装配，根据Bean的类型和当前bean的属性的类型进行自动装配，若IOC容器中有1个以上的类型匹配bean则抛异常
-->
<bean id="address" class="com.autowire.Address"
      p:city="Beijing" p:street="HuiLongGuan"/>

<bean id="car" class="com.autowire.Car"
      p:brand="Audi" p:price="300000"/>

<bean id="people" class="com.autowire.People"
      p:name="Tom" autowire="byType"/>
```

**自动装配的缺点：** 配置了 autowire 属性会装配Bean的所有属性，不能只装配个别属性；自动装配要么根据类型自动装配，要么根据名称装配，不能兼得。

- **（8）bean 之间的关系（配置继承、依赖Bean配置）：**

继承bean配置：Spring中可以继承bean的配置，被继承的bean被称为父bean，继承父bean的bean称为子bean，子bean从父bean中继承配置包括属性配置。

> 抽象 bean ：bean 的 abstract 属性为true的bean ，不能被IOC容器实例化，只用来被继承。

```xml
<!-- bean之间的配置继承 -->
<bean id="address" class="com.autowire.Address"
      p:city="Beijing" p:street="WuDaoKou"/>

<!--bean的配置继承，使用bean的parent属性指定继承哪个bean的配置-->
<bean id="address2" p:street="DaZhongSi" parent="address"/>
```

依赖bean配置： Spring 允许用户通过 depends-on 属性设置bean的前置依赖，设置依赖的bean会在本bean实例化之前创建好。

```xml
<bean id="car" class="com.autowire.Car" p:brand="Audi" p:price="300000"/>

<!--要求在配置person时，必须有一个关联的car, person这个bean依赖car这个bean-->
<bean id="person" class="com.autowire.People"
      p:name="Tom" p:address-ref="address2" depends-on="car"/>
```

- **（9）bean 的作用域：**

Spring IOC容器中的对象默认是单例的（singleton），可以通过bean的 **scope** 属性设置对象的作用域。

singleton：单例模式，不管获取多少个对象实例，都是同一个对象，容器加载时就会创建这个实例。

prototype：多例模式，每次获取的对象都是不同的，相当于每次都new了一个对象，容器初始化时不创建bean的实例，而在每次请求时创建一个新的实例并返回。

- **（10）使用外部属性文件：**

在使用数据库连接时需要将配置文件外置，通过引用外部属性文件中的值来配置bean。首先需要在 xml 文件中使用 <context:property-placeholder /> 标签引入外部文件，然后使用 ${属性名} 引用外部属性文件值。

```xml
<!--导入属性文件-->
<context:property-placeholder location="classpath:db.properties"/>

<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
    <!--使用外部属性文件的属性-->
    <property name="user" value="${user}"/>
    <property name="password" value="${password}"/>
    <property name="jdbcUrl" value="${url}"/>
    <property name="driverClass" value="${driverClass}"/>
</bean>
```

- **（11）Spring 表达式语言 SpEL：**

SpEL：Spring表达式语言，是一个支持运行时查询和操作对象图的强大的表达式语言。使用 #{...} 作为界定符，所有在大括号中的字符都将被认为是SpEL。

SpEL为 bean 的属性进行动态赋值提供了便利，通过SpEL可以实现 通过bean的id对bean进行引用；调用方法以及引用对象中的属性；计算表达式的值；匹配正则表达式。

```xml
<bean id="address" class="com.spel.Address">
    <!--使用spel为属性赋值一个字面值-->
    <property name="city" value="#{'Beijing'}"/>
    <property name="street" value="WuDaoKou"/>
</bean>

<bean id="car" class="com.spel.Car">
    <property name="brand" value="Audi"/>
    <property name="price" value="500000"/>
    <!--使用sqpl引用类的静态属性-->
    <property name="typePerimeter" value="#{T(java.lang.Math).PI * 80}"/>
</bean>

<bean id="person" class="com.spel.People">
    <!--使用spel来引用其它的bean-->
    <property name="car" value="#{car}"/>
    <!--使用spel来引用其它bean的数据-->
    <property name="city" value="#{address.city}"/>
    <!--在spel中使用运算符-->
    <property name="info" value="#{car.price > 300000 ? '金领' : '白领'}"/>
    <property name="name" value="Tom"/>
</bean>
```
- **（12）IOC容器 bean 的生命周期：**

  Spring IOC容器可以管理 bean 的生命周期，Spring 允许在 bean 生命周期的特定点执行定制的任务。

  IOC 容器管理生命周期的过程：

  - 通过构造器或工厂方法创建 bean 实例
  - 为 bean 的属性设置值和对其它 bean 的引用
  - 调用 bean 的初始化方法
  - bean 可以使用了
  - 当容器关闭时，调用 bean 的销毁方法

  在 bean 的声明里设置 init-method 和 destroy-method 属性，为 bean 指定初始化方法和销毁方法。

  ```xml
  <bean id="car" class="com.yunhui.cycle.Car" init-method="init" destroy-method="destroy">
      <property name="brand" value="Audi"/>
  </bean>
  ```

- **（13）通过工厂方法配置 bean：** 

  **静态工厂方法：** 调用静态工厂方法创建 bean 是将创建 bean 的过程封装到静态方法中，要声明通过静态方法创建 bean 需要在 bean 的 class 属性里指定拥有该工厂方法的类，同时在 factory-method 属性里指定工厂方法的名称，最后使用 \<constrctor-arg> 元素为该方法传递方法参数。

  ```java
  /**
   * 静态工厂方法：直接调用某一个类的静态方法返回bean的实例
   */
  public class StaticCarFactory {
  
      private static Map<String, Car> cars = new HashMap<>();
  
      static{
          cars.put("Audi", new Car("Audi", 300000));
          cars.put("Ford", new Car("Ford", 400000));
      }
  
      //静态工厂方法
      public static Car getCar(String name){
          return cars.get(name);
      }
  }
  ```

  ```xml
  <!--
      通过静态工厂方法来配置 bean 不是配置静态工厂方法类实例，而是配置工厂方法创建的 bean 实例
      class：指向静态工厂方法的全类名
      factory-method：指向静态工厂的方法名
      <constructor-arg value=""/>：如果工厂方法需要参数则需要使用该标签传入参数
  -->
  <bean id="car1" class="com.yunhui.factory.StaticCarFactory" factory-method="getCar">
      <constructor-arg value="Audi"/>
  </bean>
  ```

  **实例工厂方法：** 将对象的创建过程封装到另一个对象实例的方法中，要声明通过实例工厂方法创建 bean 在 bean 的 factory-bean 属性里指定拥有该工厂方法的 bean ；在 factory-method 属性里指定该工厂方法的名称；使用  \<constrctor-arg> 元素为工厂方法传递参数。

  ```java
  /**
   * 实例工厂方法：实例工厂的方法，即先需要创建工厂本身再调用工厂的实例方法来返回 bean 的实例
   */
  public class InstanceCarFactory {
  
      private Map<String, Car> cars = null;
  
      public InstanceCarFactory(){
          cars = new HashMap<>();
          cars.put("Audi", new Car("Audi", 300000));
          cars.put("Ford", new Car("Ford", 400000));
      }
  
      public Car getCar(String brand){
          return cars.get(brand);
      }
  }
  ```

  ```xml
  <!--配置工厂的实例-->
  <bean id="carFactory" class="com.factory.InstanceCarFactory"/>
  
  <!--
      通过实例工厂方法来配置bean
      factory-bean：指向实例工厂实例的 bean
      factory-method：指向工厂类创建对象的方法名
      <constructor-arg value=""/>：如果工厂方法需要参数则需要使用该标签传入参数
  -->
  <bean id="car2" factory-bean="carFactory" factory-method="getCar">
      <constructor-arg value="Ford"/>
  </bean>
  ```


#### 3、Spring Bean 配置（基于注解的方式）：

使用注解方式配置 bean 需要在classpath中开启组件扫描。开启组件扫描后Spring能够侦测和实例化具有特定注解的组件。

特定组件包括：

（1）、@Component：基本注解，标识一个受 Spring 管理的组件；

（2）、@Repository：标识持久层组件；

（3）、@Service：标识服务层组件（业务层）；

（4）、@Controller：标识表现层组件。

对于扫描到的组件，Spring 有默认的命名策略：使用非限定类名，第一个字母小写，也可以在注解中通过 value 属性标识组件的名称。

在类上使用了特定的注解后还需要在 Spring 的配置文件中声明 \<context:component-scan/> 通过 base-package 属性指定一个需要扫描的基类包，Spring 容器将会扫描这个基类包及其子类包中的所有类。需要扫描多个包时可以用逗号隔开。

如果仅希望扫描特定类而非基类包下的所有类，可以使用 resource-patten 属性来过滤指定的类；

\<context:include-filter>：子节点表示要包含的目标类；

\<context:exclude-filter>：子节点要排除在外的目标类；

\<context:component-scan/> 下可以拥有若干个 \<context:include-filter> 和 \<context:exclude-filter> 子节点。

```xml
<!--指定 Spring IOC 容器扫描包-->
<context:component-scan base-package="com.annotation"/>
```

```xml
<!--扫描目标包中只包含 @Repository 注解的类，需要配合 use-default-filters="false" 一起使用-->
<context:component-scan base-package="com.yunhui.annotation" use-default-filters="false">
    <context:include-filter type="annotation" expression="org.springframework.stereotype.Repository"/>
</context:component-scan>
```

\<context:component-scan/> 元素还会自动注册 AutowiredAnnotationBeanPostProcessor 实例，该实例可以自动装配具有 @Autowired、@Resource、@Inject注解的属性。

**@Autowired：**

- 自动装配具有兼容类型的单个 Bean 属性。可以用在 构造器、普通字段（即使是非public）、一切具有参数的方法。

  ```java
  @Autowired
  private UserRepository userRepository;
  
  @Autowired
  public void setUserRepository(UserRepository userRepository) {
      this.userRepository = userRepository;
  }
  ```

- 默认情况下所有使用 @Autowired 注解的属性都需要被设置，当Spring找不到匹配的 bean 装配属性时会抛出异常，若某一属性允许不会设置，可以设置 @Autowired 注解的 required 属性为 false。

  ```java
  //如果没有相应的 bean 则不会报错，对象显示 null
  @Autowired(required = false)
  private UserRepository userRepository;
  ```

- 默认情况下当IOC容器中存在多个类型兼容的 bean 时，通过类型的自动装配将无法工作，此时可以在 @Qualifier 注解里提供 bean 的名称，Spring 允许对方法的入参标注 @Qualifier 以指定注入 bean 的名称。

  ```java
  @Autowired
  @Qualifier("userRepository")	//通过@Qualifier注解指定是哪一个bean
  private UserRepository userRepository;
  
  @Autowired
  public void setUserRepository(@Qualifier("userRepository") UserRepository userRepository) {
      this.userRepository = userRepository;
  }
  ```

- @Autowired 注解也可以应用在 **数组** 类型的属性上，此时 Spring 读取该集合的类型信息，然后自动装配与之兼容的 bean 。

- @Autowired 注解也可以应用在 **集合** 类型的属性上，此时 Spring 读取该集合的类型信息，然后自动装配与之兼容的 bean 。

- @Autowired 注解用在 **Map** 上时，若该 map 的键值为 String 那么 Spring 将自动装配与之 Map 值兼容的 bean。

**@Resource：** 该注解要求提供一个 bean 名称的属性，若该属性为空则自动采用标注处的变量名或方法名作为 bean 的名称。

**@Inject：** 和 @Autowired 注解一样也是按照类型匹配注入的 bean 但没有 required 属性。

#### 4、泛型依赖注入：

Spring 4.x 中可以为子类注入子类对应的泛型类型的成员变量。

```java
public class BaseService<T> {

    @Autowired(required = false)
    protected BaseRepository<T> repository;

    public void add(){
        System.out.println("add...");
        System.out.println(repository);
    }
}

@Service
public class UserService extends BaseService<User> {
}

public class BaseRepository<T> {
}

@Repository
public class UserRepository extends BaseRepository<User> {
}

public class User {
}

//测试
public class SpringTest10 {
    ApplicationContext application = new ClassPathXmlApplicationContext("beans-generic.xml");

    @Test
    public void test1() {
        UserService userService = (UserService) application.getBean("userService");
        userService.add();
    }
}
```

#### 5、Spring AOP：

- **为什么使用 AOP ：** 

  代码混乱：越来越多的非业务需求（日志和验证等）加入后，原有的业务方法急剧膨胀，每个地方在处理核心逻辑的同时还需要兼顾其它多个关注点。

  代码分散：以日志需求为例，只是为了满足单一需求不得不在多个模块里重复相同的代码，如果日志需求发生变化必须修改所有的模块。

- **问题解决：** 

  AOP动态代理，使用一个代理将对象包封装起来，然后用该代理对象取代原始对象，任何原始对象的调用都要通过代理，代理对象决定是否以及何时将方法调用转到原始对象上。

- **Java动态代理：** 

  ```java
  //需求：在方法执行前进行日志输出
  public class CalculatorImpl implements Calculator {
      @Override
      public int add(int i, int j) {
          return i + j;
      }
  
      @Override
      public int sub(int i, int j) {
          return i - j;
      }
  
      @Override
      public int mul(int i, int j) {
          return i * j;
      }
  
      @Override
      public int div(int i, int j) {
          return i / j;
      }
  }
  ```

  ```java
  //创建代理类
  public class CalculatorProxy {
  
      //要代理的对象
      private Calculator target;
      //通过构造函数传入对象
      public CalculatorProxy(Calculator target){
          this.target = target;
      }
      //实现代理的方法
      public Calculator getLoggingProxy(){
          Calculator proxy = null;
          //代理对象由哪一个类加载器负责加载
          ClassLoader loader = target.getClass().getClassLoader();
          //代理对象的类型，即其中有哪些方法
          Class[] interfaces = new Class[]{Calculator.class};
          //当调用代理对象其中的方法时，执行该代码
          InvocationHandler h = new InvocationHandler() {
              /**
               * @param proxy 正在返回的代理对象，一般情况下在 invoke 方法中都不使用该对象
               * @param method 正在被调用的方法
               * @param args 调用方法时，传入的参数
               */
              @Override
          public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                  //日志输出
                  System.out.println("The method " + method.getName() + " begin with " + Arrays.asList(args));
                  //执行方法
                  Object object = method.invoke(target, args);
                  //日志输出
                  System.out.println("The method " + method.getName() + " end with " + object);
                  return object;
              }
          };
  
          proxy = (Calculator) Proxy.newProxyInstance(loader, interfaces, h);
          return proxy;
      }
  }
  ```

  ```java
  //测试
  @Test
  public void test1(){
      Calculator c1 = new CalculatorImpl();	//创建原有的类对象
      //创建代理对象，将原有对象作为参数传入，并返回代理对象
      Calculator proxy = new CalculatorProxy(c1).getLoggingProxy();
      //通过代理对象调用方法
      System.out.println(proxy.add(1, 2));
      System.out.println(proxy.sub(3, 2));
  }
  ```

- **Spring AOP：**

  > AOP 面向切面编程，是一种新的方法论，是对传统OOP思想的补充。AOP 的主要编程对象是切面（aspect），即切面是横切关注点的模块化。切面中是横切关注点对应的方法。
  >
  > 在应用 AOP 编程时仍然需要定义公共功能，但可以明确的定义这个功能在哪里，以什么方式应用，并且不修改受影响的类，这样一来横切关注点就被模块化到特殊的对象（切面）里。

  **AOP术语：** 

  1、 **切面（Aspect）：** 横切关注点被模块化的特殊对象；Joinpoint + Advice（去哪些地方 在什么时机 做哪些增强）

  2、 **通知（Advice）：** 切面必须要完成的工作；增强，拦截到 Joinpoint 之后在方法执行的某一个时机要做怎样的增强（WHEN、WHAT）。

  3、 **目标（Target）：** 被通知的对象；

  4、 **代理（Proxy）：** 向目标对象应用通知后创建的对象；

  5、 **连接点（Joinpoint）：** 程序执行的某个特定位置，如方法执行前、执行后等。连接点由两个信息确定：方法，表示的程序执行点；相对点，表示方位（方法执行前还是后）。需要被增强的方法（WHERE）。

  6、 **切点（pointcut）：** 每个类都拥有多个连接点，即连接点是程序类中客观存在的事务，AOP通过切点定位到特定的连接点。类比：连接点相当于数据库中的记录，切点相当于查询条件。切点和连接点不是一对一的关系，一个切点匹配多个连接点。切入点相当于连接点（Joinpoint）的集合

- **AspectJ：** Java社区里最完整最流行的AOP框架。

  1、添加 AspectJ 类库；

  ```xml
  <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-aop</artifactId>
      <version>5.1.2.RELEASE</version>
  </dependency>
  <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-aspects</artifactId>
      <version>5.1.2.RELEASE</version>
  </dependency>
  <dependency>
      <groupId>aopalliance</groupId>
      <artifactId>aopalliance</artifactId>
      <version>1.0</version>
  </dependency>
  <dependency>
      <groupId>org.aspectj</groupId>
      <artifactId>aspectjweaver</artifactId>
      <version>1.9.2</version>
  </dependency>
  ```

  2、将 AOP Schema 添加到 \<beans> 根元素中（命名空间）；

  ```xml
  <beans xmlns="http://www.springframework.org/schema/beans"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns:context="http://www.springframework.org/schema/context"
         xmlns:aop="http://www.springframework.org/schema/aop"
         xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">
  </beans>
  ```

  3、在Spring中启用 AspectJ 注解支持：在 IOC 容器中启用 AspectJ 注解支持 \<aop:aspectj-aotoproxy>；

  ```xml
  <!--配置自动扫描的包-->
  <context:component-scan base-package="com.yunhui.aop.impl"/>
  <!--使 AspectJ 注解起作用，自动为匹配的类生成代理对象-->
  <aop:aspectj-autoproxy/>
  ```

  4、配置切面类（基于注解的方式）；

  ```java
  /**
   * 把该类声明为一个切面：把该类放到 IOC 容器中、再声明为一个切面
   */
  @Component
  @Aspect
  public class LoggingAspect {
  
      /**
       * 声明该方法是一个前置通知：在目标方法开始之前执行
       * @Before 说明这是一个前置通知，方法执行之前执行
       * execution 表达式是告诉aop该方法在哪个方法前执行
       */
      @Before("execution(public int com.aop.impl.Calculator.*(..))")
      public void beforeMethod(){
          System.out.println("The method begins");
      }
  }
  ```

  **5、利用方法签名编写AspectJ切入点表达式：**

  ```java
  execution(<修饰符>? <返回值类型> <声明类型>? <方法名>(<参数>) <异常>?)
  
  public static Class java.lang.Class.forName(String className) throw ClassNotFoundException;
  
  execution(* com.yunhui.aop.impl.Calculator.*(..))
  ```

  表达式中第一个 * 表示任意修饰符以及任意返回值；第二个 * 表示任意方法；括号中的 .. 表示任意数量的参数。

  6、利用 JoinPoint 作为参数访问方法的细节：

  ```java
  @Before("execution(* com.aop.impl.Calculator.*(..))")
  public void beforeMethod(JoinPoint joinPoint){
      String methodName = joinPoint.getSignature().getName();
      List<Object> args = Arrays.asList(joinPoint.getArgs());
      System.out.println("The method " + methodName + " begins with " + args);
  }
  ```

  joinPoint.getSignature().getName()：获取方法的方法名；

  joinPoint.getArgs()：获取方法的参数信息

- **通知类型：**

  **前置通知：** 连接点执行前执行的通知；

  ```java
  /**
   * 声明该方法是一个前置通知：在目标方法开始之前执行
   */
  @Before("execution(* com.aop.impl.Calculator.*(..))")
  public void beforeMethod(JoinPoint joinPoint){
      String methodName = joinPoint.getSignature().getName();
      List<Object> args = Arrays.asList(joinPoint.getArgs());
      System.out.println("The method " + methodName + " begins with " + args);
  }
  ```

  **后置通知：** 连接点完成执行之后执行的通知，即连接点返回结果或抛出异常后执行的通知。

  ```java
  /**
   * 后置通知：在方法执行之后执行的通知（无论是否发生异常）
   * 后置通知中不能访问目标方法执行的结果
   */
  @After("execution(* com.aop.impl.Calculator.*(..))")
  public void afterMethod(JoinPoint joinPoint){
      String methodName = joinPoint.getSignature().getName();
      System.out.println("The method " + methodName + " end");
  }
  ```

  **返回通知：** 在程序正常结束后执行的代码

  ```java
  /**
   * 返回通知，在程序正常结束后执行的代码
   * 返回通知是可以访问到方法的返回值的
   */
  @AfterReturning(value = "execution(* com.aop.impl.Calculator.*(..))", returning = "result")
  public void afterReturning(JoinPoint joinPoint, Object result) {
      String methodName = joinPoint.getSignature().getName();
      System.out.println("The method " + methodName + " end with " + result);
  }
  ```

  **异常通知：** 在方法出现异常时会执行的代码，可以访问到异常对象，且可以指定在出现特定异常时再执行通知代码

  ```java
  /**
   * 异常通知，在方法出现异常时会执行的代码
   * 可以访问到异常对象，且可以指定在出现特定异常时再执行通知代码
   */
  @AfterThrowing(value = "execution(* com.aop.impl.Calculator.*(..))", throwing = "ex")
  public void afterThrowing(JoinPoint joinPoint, Exception ex) {
      System.out.println("The method " + joinPoint.getSignature().getName() + " occurs exception:" + ex);
  }
  ```

  **环绕通知：**

  ```java
  /**
   * 环绕通知需要携带 ProceedingJoinPoint 类型的参数
   * 环绕通知类似于动态代理的全过程：ProceedingJoinPoint 可以决定是否执行目标方法
   * 环绕通知必须要有返回值，返回值即目标方法的返回值
   */
  @Around("execution(* com.aop.impl.Calculator.*(..))")
  public Object aroundMethod(ProceedingJoinPoint pjd){
      System.out.println("aroundMethod");
      Object result = null;
      String methodName = pjd.getSignature().getName();
      try {
          System.out.println("The method " + methodName + " begin with " + Arrays.asList(pjd.getArgs()));
          result = pjd.proceed();
          System.out.println("The method " + methodName + " end with " + result);
      } catch (Throwable throwable) {
          throwable.printStackTrace();
      }
      return result;
  }
  ```

- **切面的优先级：**

  配置多个切面后，当需要切面按照一定顺序执行就需要配置切面的优先级。在切面类上加注解 **@Order(数值)** 其中数值越小优先级越高

- **重用切点表达式：**

  ```java
  @Order(1)//配置优先级，数值越小优先级越高
  @Component
  @Aspect
  public class LoggingAspect {
  
      /**
       * 定义一个方法用于声明切入点表达式，一般的该方法中不需要填入其它的代码
       * 内部引用该表达式，直接写 方法名() 即可
       * 外部引用该表达式（同包）， 本类名.方法名()
       * 不同包下引用还需要加上包名
       */
      @Pointcut("execution(* com.aop.impl.Calculator.*(..))")
      public void declareJoinPoint(){}
      
      /**
       * 声明该方法是一个前置通知：在目标方法开始之前执行
       */
      @Before("declareJoinPoint()")
      public void beforeMethod(JoinPoint joinPoint) {
          String methodName = joinPoint.getSignature().getName();
          List<Object> args = Arrays.asList(joinPoint.getArgs());
          System.out.println("The method " + methodName + " begins with " + args);
      }
  
      /**
       * 后置通知：在方法执行之后执行的通知（无论是否发生异常）
       * 后置通知中不能访问目标方法执行的结果
       */
      @After("declareJoinPoint()")
      public void afterMethod(JoinPoint joinPoint) {
          String methodName = joinPoint.getSignature().getName();
          System.out.println("The method " + methodName + " end");
      }
  
      /**
       * 返回通知，在程序正常结束后执行的代码
       * 返回通知是可以访问到方法的返回值的
       */
      @AfterReturning(value = "declareJoinPoint()", returning = "result")
      public void afterReturning(JoinPoint joinPoint, Object result) {
          String methodName = joinPoint.getSignature().getName();
          System.out.println("The method " + methodName + " end with " + result);
      }
  
      /**
       * 异常通知，在方法出现异常时会执行的代码
       * 可以访问到异常对象，且可以指定在出现特定异常时再执行通知代码
       * @param joinPoint
       * @param ex
       */
      @AfterThrowing(value = "declareJoinPoint()", throwing = "ex")
      public void afterThrowing(JoinPoint joinPoint, Exception ex) {
          System.out.println("The method " + joinPoint.getSignature().getName() + " occurs exception:" + ex);
      }
  }
  ```

- **基于XML文件配置：**

  ```xml
  <!--配置bean-->
  <bean id="calculator" class="com.aop.xml.CalculatorImpl"/>
  
  <!--WHAT做什么增强，通知类Advice-->
  <bean id="logging" class="com.aop.xml.LoggingAspect"/>
  
  <!--配置AOP-->
  <aop:config proxy-target-class="true"> <!--使用CGLIB动态代理-->
      <!--配置切点表达式-->
      <aop:pointcut id="pointcut" expression="execution(* com.aop.xml.Calculator.*(..))"/>
      <!--配置切面 ref关联WHAT-->
      <aop:aspect ref="logging" order="1">
          <aop:before method="beforeMethod" pointcut-ref="pointcut"/>
          <aop:after method="afterMethod" pointcut-ref="pointcut"/>
          <aop:after-returning method="afterReturning" pointcut-ref="pointcut" returning="result"/>
      </aop:aspect>
  </aop:config>
  ```

#### 6、Spring  jdbcTemplate：

- **Spring jdbcTemplate：** Spring 为简化持久化操作，在 JDBC API 的基础上技工了 JDBC Template 组件

  1、配置数据源：

  ```xml
  <!--配置数据源-->
  <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
      <property name="user" value="root"/>
      <property name="password" value="root"/>
      <property name="jdbcUrl" value="jdbc:mysql://localhost:3306/jdbc_db"/>
      <property name="driverClass" value="com.mysql.jdbc.Driver"/>
  </bean>
  ```

  2、配置jdbcTemolate：

  ```xml
  <!--配置 jdbcTemplate-->
  <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
      <property name="dataSource" ref="dataSource"/>
  </bean>
  ```

  **3、使用jdbcTemplate进行数据库操作：**

  ```java
  @ContextConfiguration("classpath:applicationContext.xml")
  @RunWith(SpringJUnit4ClassRunner.class)
  public class jdbc_test1 {
  
      @Autowired
      JdbcTemplate jdbcTemplate;
  
      /**
       * 执行 INSERT UPDATE DELETE 语句
       */
      @Test
      public void testUpdate(){
          String sql = "update student set sname = ? where sid = ?";
          jdbcTemplate.update(sql,"AA",4);
      }
  
      /**
       * 执行批量更新
       */
      @Test
      public void testBatchUpdate(){
          String sql = "insert into student values(null,?,?)";
          List<Object[]> batchArgs = new ArrayList<>();
          batchArgs.add(new Object[]{"AA",1});
          batchArgs.add(new Object[]{"BB",2});
          batchArgs.add(new Object[]{"CC",3});
          batchArgs.add(new Object[]{"DD",4});
          batchArgs.add(new Object[]{"EE",3});
          jdbcTemplate.batchUpdate(sql, batchArgs);
      }
  
      /**
       * 从数据库获取一条记录，得到对应的对象
       * 需要使用 queryForObject(String sql, RowMapper<T> rowMapper, @Nullable Object... args)
       * 其中 RowMapper 指定如何去映射结果集的行，常用的实现类为 BeanPropertyRowMapper
       * 使用 SQL 中列的别名完成列名和属性名的映射
       * 不支持级联属性
       */
      @Test
      public void testQueryForObject(){
          String sql = "select sid, sname, sclazz from student where sid = ?";
          RowMapper<Student> rowMapper = new BeanPropertyRowMapper<>(Student.class);
          Student student = jdbcTemplate.queryForObject(sql, rowMapper, 1);
          System.out.println(student);
      }
  
      /**
       * 查询实体类的集合
       * 调用的不是 QueryForList
       */
      @Test
      public void testQueryForList(){
          String sql = "select sid, sname, sclazz from student";
          RowMapper<Student> rowMapper = new BeanPropertyRowMapper<>(Student.class);
          List<Student> student = jdbcTemplate.query(sql, rowMapper);
          System.out.println(student);
      }
  
      /**
       * 获取单列的值或统计查询
       */
      @Test
      public void queryForObject2(){
          String sql = "select count(sid) from student";
          long count = jdbcTemplate.queryForObject(sql, Long.class);
          System.out.println(count);
      }
  }
  ```

- **JdbcDaoSupport进行数据库操作（不推荐使用）：**

  每个dao类继承 JdbcDaoSupport 类，并设置 jdbcTemplate 。在类中使用 jdbcTemplate 进行数据库操作。

  ```java
  //继承 JdbcDaoSupport
  public class StudentDaoImpl extends JdbcDaoSupport {
  
      public Student getStudent(int sid){
          String sql = "select * from student where sid = ?";
          return getJdbcTemplate().queryForObject(sql, new BeanPropertyRowMapper<>(Student.class), sid);
      }
  }
  ```

- **使用具名参数：**

  具名参数：可以为参数起名字，使用具名参数增强了代码的可维护性，不用再去对应位置，直接对应参数名。

  1、配置 NamedParameterJdbcTemplate :

  ```xml
  <!--配置NamedParameterJdbcTemplate，该对象可以使用具名参数，其没有无参构造器，必须为其构造器指定参数-->
  <bean id="namedParameterJdbcTemplate" class="org.springframework.jdbc.core.namedparam.NamedParameterJdbcTemplate">
      <constructor-arg ref="dataSource"/>
  </bean>
  ```

  2、使用具名参数：

  ```java
  @ContextConfiguration("classpath:applicationContext.xml")
  @RunWith(SpringJUnit4ClassRunner.class)
  public class jdbc_test1 {
  
      @Autowired
      NamedParameterJdbcTemplate namedParameterJdbcTemplate;
  
      /**
       * 使用具名参数时，可以使用 update(String sql, SqlParameterSource paramSource) 进行更新操作
       * 可以传入封装好的对象
       */
      @Test
      public void test_NamedParameterJdbcTemplate2(){
          String sql = "insert into student values(:sid, :sname, :sclazz)";
          Student student = new Student();
          student.setSid(null);
          student.setSname("BB");
          student.setSclazz(2);
          SqlParameterSource sqlParameterSource = new BeanPropertySqlParameterSource(student);
          namedParameterJdbcTemplate.update(sql, sqlParameterSource);
      }
  
      /**
       * 测试具名参数，可以为参数起名字
       * 具名参数增强了可维护性，不会再去对应位置，直接对应参数名
       */
      @Test
      public void test_NamedParameterJdbcTemplate(){
          String sql = "insert into student values(null, :sname, :sclazz)";
          Map<String, Object> paramMap = new HashMap<>();
          paramMap.put("sname", "AA");
          paramMap.put("sclazz", 3);
          namedParameterJdbcTemplate.update(sql, paramMap);
      }
  }
  ```

#### 7、Spring 事务管理：

事务是用来保证数据的完整性和一致性。事务就是一系列的动作，他们被当做一个单独的工作单元，**这个动作要么全部成功，要么全部失败。** 

事务的关键属性（ACID）：

1、**原子性：**事务是一个原子操作，有一系列动作组成，事务的原子性确保动作要么全部成功，要么全部失败；

2、**一致性：**一旦所有事务完成，事务就被提交，数据和资源就处于一种满足业务规则的一致性状态中；

3、**隔离性：**可能有多个事务同时处理相同的数据，每个事务都应该与其它事务隔离开，防止数据损坏；

4、**持久性：**一旦事务完成，无论发生什么系统错误，它的结果都不应该受到影响。

- **案例：**

  买书案例，购买书籍，一次购买一本，购买成功书籍库存减一，用户账户减去相应的价钱。

  ```java
  //书籍Dao接口
  public interface BookShopDao {
  
      /**
       * 根据书号获取书的单价
       * @param isbn 书号
       * @return 书的单价
       */
      int findBookPriceByIsbn(String isbn);
  
      /**
       * 更新书的库存，使书号对应的库存减一
       * @param isbn 书号
       */
      void updateBookStock(String isbn);
  
      /**
       * 更新用户的账户余额，username的balance - price
       * @param username 用户名
       * @param price 书的价格
       */
      void updateUserAccount(String username, int price);
  }
  ```

  ```java
  //书籍实现类
  @Repository("bookShopDao")
  public class BookShopDaoImpl implements BookShopDao {
  
      @Autowired
      private JdbcTemplate jdbcTemplate;
      
      @Override
      public int findBookPriceByIsbn(String isbn) {
          String sql = "select price from spring_book where isbn = ?";
          return jdbcTemplate.queryForObject(sql, Integer.class, isbn);
      }
      
      @Override
      public void updateBookStock(String isbn) {
          //检查书的库存是否足够，若不够则抛异常
          String sql2 = "select stock from spring_stock where isbn = ?";
          int stock = jdbcTemplate.queryForObject(sql2, Integer.class, isbn);
          if (stock == 0) {
              throw new BookStockException("库存不足");//自定义异常类
          }
          String sql = "update spring_stock set stock = stock - 1 where isbn = ?";
          jdbcTemplate.update(sql,isbn);
      }
  
      @Override
      public void updateUserAccount(String username, int price) {
          //检查书的余额是否足够，若不够则抛异常
          String sql2 = "select balance from spring_account where username = ?";
          int balance = jdbcTemplate.queryForObject(sql2, Integer.class, username);
          if (balance < price) {
              throw new UserAccountException("余额不足");//自定义异常类
          }
          String sql = "update spring_account set balance = balance - ? where username = ?";
          jdbcTemplate.update(sql, price, username);
      }
  }
  ```

  ````java
  //购买书籍Service接口
  public interface BookShopService {
      void purchase(String username, String isbn);
  }
  ````

  ```java
  //购买书籍Service实现类
  @Service
  public class BookShopServiceImpl implements BookShopService {
  
      @Autowired
      private BookShopDao bookShopDao;
  
      @Override
      public void purchase(String username, String isbn) {
          //1、获取数的单价
          int price = bookShopDao.findBookPriceByIsbn(isbn);
          //2、更新书的库存
          bookShopDao.updateBookStock(isbn);
          //3、更新用户余额
          bookShopDao.updateUserAccount(username, price);
      }
  }
  ```

  ```java
  //测试
  @RunWith(SpringJUnit4ClassRunner.class)
  @ContextConfiguration("classpath:beans-tx.xml")
  public class SpringTransactionTest {
  
      @Autowired
      private BookShopService bookShopService;
  
      @Test
      public void testBookShopService(){
          bookShopService.purchase("AA", "1001");
      }
  }
  ```

  存在问题：购买书籍会有更改书籍库存、减掉用户金额的操作，在没有加入事务管理时，这两步操作存在风险，有可能会导致更新书籍库存后因意外情况导致用户金额没有减掉，从而造成数据混乱。

- **Spring声明式事务（基于注解的形式）：**

  1、配置事务管理器 DataSourceTransactionManager 需要配置数据源

  ```xml
  <!--配置事务事务管理器-->
  <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
      <property name="dataSource" ref="dataSource"/>
  </bean>
  ```

  2、开启事务注解，xml 配置文件需要导入事务管理 tx 命名空间

  ```xml
  <!--启用事务注解-->
  <tx:annotation-driven transaction-manager="transactionManager"/>
  ```

  3、使用 **@Transactional** 注解进行事务管理

  ```java
  @Service
  public class BookShopServiceImpl implements BookShopService {
  
      @Autowired
      private BookShopDao bookShopDao;
  
      //添加事务注解
      @Transactional
      @Override
      public void purchase(String username, String isbn) {
          //1、获取数的单价
          int price = bookShopDao.findBookPriceByIsbn(isbn);
          //2、更新书的库存
          bookShopDao.updateBookStock(isbn);
          //3、更新用户余额
          bookShopDao.updateUserAccount(username, price);
      }
  }
  ```

- **事务的传播行为：**

  当事务方法被另一个事务方法调用时， **必须指定事务应该如何传播** 。例如：方法可以继续在现有事务中运行，也可以开启一个新的事务并在自己的事务中运行。事务的传播属性可以在 @Transactional 注解的 propagation 属性中定义。

  1、 **REQUIRED：** 如果有事务在运行，当前事务就在这个事务内运行，否则就启动一个新的事务，并在自己的事务内运行。（ **事务的默认传播行为** ）

  2、 **REQUIRES_NEW：** 当前的方法必须启动新事务并在它自己的事务中运行，如果有事务正在运行，应该将它挂起。

  ```java
  //接口
  public interface Cashier {
      void checkout(String username, List<String> isbns);
  }
  ```

  ```java
  //实现类
  @Service("cashier")
  public class CashierImpl implements Cashier {
  
      @Autowired
      private BookShopService bookShopService;
  
      //购买多本书籍，该方法作为一个事务调用另一个方法事务
      @Transactional
      @Override
      public void checkout(String username, List<String> isbns) {
          for (String isbn : isbns) {
              bookShopService.purchase(username, isbn);//该方法也是一个事务行为
          }
      }
  }
  ```

  ```java
  @Service
  public class BookShopServiceImpl implements BookShopService {
  
      @Autowired
      private BookShopDao bookShopDao;
  
      //添加事务注解，使用 propagation 指定事务的传播行为。
      //当前事务方法在被另外一个事务调用时该如何使用事务，默认取值为 REQUIRED 使用调用方法的事务
      @Transactional(propagation = Propagation.REQUIRED)
      @Override
      public void purchase(String username, String isbn) {
          //1、获取数的单价
          int price = bookShopDao.findBookPriceByIsbn(isbn);
          //2、更新书的库存
          bookShopDao.updateBookStock(isbn);
          //3、更新用户余额
          bookShopDao.updateUserAccount(username, price);
      }
  }
  ```

  运行结果：内部方法会使用外部方法的事务，相当于外部方法的事务是一个整体，如果执行过程中抛出异常，则程序会将数据回滚至初始状态。即使第一次能够购买成功也会回滚。

  ```java
  @Service
  public class BookShopServiceImpl implements BookShopService {
  
      @Autowired
      private BookShopDao bookShopDao;
  
      //添加事务注解，使用 propagation 指定事务的传播行为。
      //不论怎样，执行该方法时都会重新开启一个事务，将调用改方法的事务挂起。
      @Transactional(propagation = Propagation.REQUIRES_NEW)
      @Override
      public void purchase(String username, String isbn) {
          //1、获取数的单价
          int price = bookShopDao.findBookPriceByIsbn(isbn);
          //2、更新书的库存
          bookShopDao.updateBookStock(isbn);
          //3、更新用户余额
          bookShopDao.updateUserAccount(username, price);
      }
  }
  ```

  运行结果：方法新开事务，会将调用方法的事务挂起，并执行自己的事务。如果第一本书籍购买成功，在第二件书籍购买失败抛出异常后购买成功的数据不会回滚。

- **并发事务所导致的问题：**

  当同一个应用程序或者不同应用程序中的多个事务在同一个数据集上并发执行时，会出现很多问题：

  1、 **脏读：** 对于两个事务 T1、T2，T1读取了已经被T2更新但还没有提交的数据，之后若 T2 回滚数据，T1 读取的内容就是临时且无效的。

  2、 **不可重复读：** 对于两个事务 T1、T2，T1 读取了一个字段，然后 T2 更新了该字段，之后T1再去读取同一个字段值就不同了。

  3、 **幻读：** 对于两个事务T1、T2，T1从表中读取了一个字段，然后T2在该表中插入一些新的行，如果T1再次读取同一个表就会多出几行。

  ```java
  //使用 isolation 指定事务的隔离级别，最常用的取值为 READ_COMMITTED
  //默认情况下 Spring 的声明式事务对所有的运行时异常进行回滚，也可以通过对应的属性极性设置
  @Transactional(propagation = Propagation.REQUIRES_NEW, isolation = Isolation.READ_COMMITTED)
  @Override
  public void purchase(String username, String isbn) {
      //1、获取数的单价
      int price = bookShopDao.findBookPriceByIsbn(isbn);
      //2、更新书的库存
      bookShopDao.updateBookStock(isbn);
      //3、更新用户余额
      bookShopDao.updateUserAccount(username, price);
  }
  ```

- **Spring声明式事务（基于XML的形式）：**

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns:tx="http://www.springframework.org/schema/tx"
         xmlns:aop="http://www.springframework.org/schema/aop"
         xsi:schemaLocation="http://www.springframework.org/schema/beans
         http://www.springframework.org/schema/beans/spring-beans.xsd
         http://www.springframework.org/schema/tx
         http://www.springframework.org/schema/tx/spring-tx.xsd
         http://www.springframework.org/schema/aop
         http://www.springframework.org/schema/aop/spring-aop.xsd">
  
      <!--配置数据源-->
      <bean id="dataSource_xml" class="com.mchange.v2.c3p0.ComboPooledDataSource">
          <property name="driverClass" value="com.mysql.jdbc.Driver"/>
          <property name="jdbcUrl" value="jdbc:mysql:///jdbc_db"/>
          <property name="user" value="root"/>
          <property name="password" value="root"/>
      </bean>
  
      <!--配置jdbcTemplate-->
      <bean id="jdbcTemplate_xml" class="org.springframework.jdbc.core.JdbcTemplate">
          <property name="dataSource" ref="dataSource_xml"/>
      </bean>
  
      <!--配置bean-->
      <bean id="bookShopDao_xml" class="com.spring.xml.BookShopDaoImpl">
          <property name="jdbcTemplate" ref="jdbcTemplate_xml"/>
      </bean>
      <bean id="bookShopService_xml" class="com.spring.xml.service.impl.BookShopServiceImpl">
          <property name="bookShopDao" ref="bookShopDao_xml"/>
      </bean>
      <bean id="cashier_xml" class="com.spring.xml.service.impl.CashierImpl">
          <property name="bookShopService" ref="bookShopService_xml"/>
      </bean>
  
      <!--1、配置事务管理器-->
      <bean id="transactionManager_xml" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
          <property name="dataSource" ref="dataSource_xml"/>
      </bean>
  
      <!--2、配置事务增强属性-->
      <tx:advice id="txAdvice_xml" transaction-manager="transactionManager_xml">
          <tx:attributes>
              <!--根据方法名指定事务的属性-->
              <tx:method name="purchase" propagation="REQUIRES_NEW"/>
              <tx:method name="*"/>
          </tx:attributes>
      </tx:advice>
  
      <!--3、配置事务切如入点，把事务切入点和事务属性关联起来-->
      <aop:config>
          <aop:pointcut id="pointcut_xml"
                        expression="execution(* com.spring.xml.service.*.*(..))"/>
          <aop:advisor advice-ref="txAdvice_xml" pointcut-ref="pointcut_xml"/>
      </aop:config>
  </beans>
  ```

  **总结：** 正常配置 数据源、jdbcTemplate、bean等，然后配置事务管理器，再配置事务属性，最后通过 AOP 思想指定事务的切面，将事务切点和事务的属性关联起来。在配置过程中所有配置全部正确，程序运行报错，排查后是少了 **aspectjweaver** 依赖，在基于注解的事务管理中并没有该 jar 包，运行正常，基于 XML 配置中需要添加该依赖。