## 一、SpringBoot 入门

#### 1、SpringBoot简介：

> 简化Spring应用开发的一个框架，是对整个 Spring 技术栈的大整合，是J2EE的一站式解决方案。

#### 2、微服务：

微服务：是一种架构风格，一个应用应该是一组小型服务；可以通过 HTTP 的方式进行互通。每一个功能元素最终都是一个可独立替换和升级的软件单元。

单体应用： ALL IN ONE 所有的功能都在一个应用中。

[微服务中文文档](http://blog.cuicc.com/blog/2015/07/22/microservices/) 

#### 3、环境准备：

SpringBoot 1.5.19          maven 3.6

#### 4、创建SpringBoot工程 Hello Word（手动创建方式）：

- **1、 创建 Maven 工程**

- **2、引入 SpringBoot 依赖**

    ```xml
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.9.RELEASE</version>
    </parent>
    
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>
    ```

- **3、编写主程序：启动 SpringBoot 应用：**

    ```java
    /**
     * @SpringBootApplication 用来标注一个主程序类，说明这是一个 SpringBoot 应用
     */
    @SpringBootApplication
    public class HelloWordApplication {
    
        public static void main(String[] args) {
            //启动 SpringBoot 应用
            SpringApplication.run(HelloWordApplication.class, args);
        }
    }
    ```

- **4、编写相关的业务逻辑代码：**

    ```java
    @Controller
    public class HelloController {
    
        @RequestMapping("/hello")
        @ResponseBody
        public String hello(){
            return "Hello Spring Boot";
        }
    }
    ```

- **5、启动主程序，直接运行 main 方法即可**

- **6、简化部署：** 添加SpringBoot的maven工具，可以将SpringBoot应用打包成一个jar包在有JDK运行环境中使用 java -jar 命令运行，不需要依赖 Tomcat 

    ```xml
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
    ```

#### 5、Hello Word 细节解析：

- **pom.xml ：** 

    ```xml
    <!-- 父项目 -->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.9.RELEASE</version>
    </parent>
    
    <!-- spring-boot-starter-parent 的父项目 真正管理 Spring Boot 应用里所有的依赖版本 -->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-dependencies</artifactId>
        <version>1.5.9.RELEASE</version>
        <relativePath>../../spring-boot-dependencies</relativePath>
    </parent>
    ```

    `spring-boot-dependencies` 是 SpringBoot 的版本仲裁中心，以后导入依赖默认是不需要写版本的（没有在 dependencies 中管理的 jar 包必须生命版本号）。

- **启动器：** 

    ```xml
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>
    ```

    `spring-boot-starter` ： spring-boot 场景启动器；帮我们导入了 web 模块正常运行所依赖的组件

    Spring Boot 将所有的功能场景都抽取出来，做成一个个的 starters （启动器），只需要在项目中引入这些 starter ，场景相关的所有依赖都会导入进来。要用什么功能就导入什么场景的启动器。

- **主程序类、主入口类：** 

    ```java
    @SpringBootApplication
    public class HelloWordApplication {
    
        public static void main(String[] args) {
            //启动 SpringBoot 应用
            SpringApplication.run(HelloWordApplication.class, args);
        }
    }
    ```

    `@SpringBootApplication` Spring Boot 应用，将该注解标注在某一个类上说明这个类是 SpringBoot 的主配置类，运行这个类的main 方法就可以启动 SpringBoot 的应用。

    `@SpringBootApplication` 由多个注解组合而成：

    ```java
    @Target(ElementType.TYPE)
    @Retention(RetentionPolicy.RUNTIME)
    @Documented
    @Inherited
    @SpringBootConfiguration
    @EnableAutoConfiguration
    @ComponentScan(excludeFilters = {
    		@Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
    		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
    ```

    **@SpringBootConfiguration：** Spring Boot 的配置类，标注在某个类上表示这个类是 Spring Boot 的配置类。

    - **@Configuration：** 配置类上需要标注这个注解。配置类也是容器中的一个组件 @Component

    **@EnableAutoConfiguration：** 开启自动配置功能；Spring Boot 自动帮我们完成自动配置

    - ```java
        @SuppressWarnings("deprecation")
        @Target(ElementType.TYPE)
        @Retention(RetentionPolicy.RUNTIME)
        @Documented
        @Inherited
        @AutoConfigurationPackage
        @Import(EnableAutoConfigurationImportSelector.class)
        ```

        **@AutoConfigurationPackage：** 自动配置包，使用了 Spring 底层注解 `@Import(AutoConfigurationPackages.Registrar.class)` 给容器中导入一个组件；导入的组件由Registrar.class指定。

        > **@AutoConfigurationPackage：** 主要作用是将主配置类（@SpringBootApplication标注的类）所在的包及其下的所有包的所有组件扫描到 Spring 容器中

        **@Import(EnableAutoConfigurationImportSelector.class)：** 给容器中导入组件，导入哪些组件的选择器；将所有需要导入的组件以全类名的形式返回，这些组件就会添加到组件中，会给组件中导入非常多的自动配置类（xxxAutoConfiguration）；给容器中导入这个场景需要的所有组件，并配置好这些组件。

        AutoConfigurationImportSelector 类中 selectImports() 方法通过 getCandidateConfigurations() 方法再调用 SpringFactoriesLoader.loadFactoryNames(EnableAutoConfiguration.class, classLoader)；加载组件。

        Spring Boot 在启动的时候从类路径下的 META-INF/spring.factories 中获取 EnableAutoConfiguration 指定的值，作为自动配置类导入到容器中，自动配置类就生效，帮我们进行自动配置工作。

#### 6、使用 Spring Initializr 快速创建 Spring Boot 项目：

在 IDEA 中创建工程时选择 Spring Initializr 创建工程，根据向导步骤修改项目名，包名等，然后选择工程需要的启动器，在这个页面可以选择需要创建的 Spring Boot 工程的版本，最后向导会联网下载创建 Spring Boot 项目。

默认生成的 Spring Boot 项目：

- 主程序已经自动生成，只需要编写自己的逻辑代码即可
- resources 文件夹中目录构成
    - static： 保存所有的静态资源；js，css，images
    - templates： 保存所有的模板页面；Spring Boot 默认jar包使用嵌入式的Tomcat，默认不支持JSP页面，可以使用模板引擎（freemarker，thymeleaf）
    - application.properties： Spring Boot应用的配置文件；可以修改一些默认设置

## 二、Spring Boot 的配置

#### 1、配置文件：

Spring Boot 使用一个全局配置的配置文件，application.properties / application.yml；配置文件放在 `src/main/resources` 目录或者 `类路径/config` 下。配置文件名是固定的。

配置文件的作用：修改 Spring Boot 自动配置的默认值。

.yml： YAML（YAML Ain't Markup Language），以数据为中心

```yaml
server:
  port: 8081
```

#### 2、YAML语法：

- **基本语法：** 

    使用 `key: value` 来表示一对键值对（注意：冒号后面必须要有 **空格** ）；

    以 **空格** 的缩进来控制层级关系，只要是左对齐的一列数据都是同一个层级的；

    ```yaml
    server:
        port: 8081
        path: /hello
    ```

    属性和值是大小写敏感的。

- **值的写法：** 

    1）、字面量：普通值（数字，字符串）：

    `key: value` 字面值直接来写，字符串默认不用加上单引号或者双引号

    ""： 双引号，不会转义字符串里的特殊字符；特殊字符会作为本身想表示的意思

    ​        name: "zhangsan \n lisi" ----------> 输出：zhangsan 换行 lisi

    ''： 单引号，会转义特殊字符，特殊字符最终只是一个普通的字符串

    ​        name: "zhangsan \n lisi" ----------> 输出：zhangsan \n lisi

    2）、对象、Map（属性和值、键值对）：

    对象还是 `key： value` 的形式，在对象的下一行来写对象的属性和值，注意缩进

    ```yaml
    friends:
    	name: zhangsan
    	age: 20
    ```

    行内写法：

    ```yaml
    friends: {name: zhangsan,age: 20}
    ```

    3）、数组（List、Set）：

    使用 `- 值` 表示数组中的一个元素

    ```yaml
    pets:
    	- cat
    	- dog
    	- pig
    ```

    行内写法：

    ```yaml
    pets: [cat,dog,pig]
    ```

#### 3、配置文件值注入：

配置文件 application.yml：

```yaml
person:
  name: zhangsan
  age: 20
  boss: false
  birth: 2017/12/15
  maps: {k1: v1, k2: 12}
  lists:
    - lisi
    - zhaoliu
  dog:
    name: 小狗
    age: 2
```

javaBean：需要提供 setter 方法才能注入属性值：

```java
/**
 * 将配置文件中配置的每一个属性的值，映射到这个组件中
 * @ConfigurationProperties: 告诉 SpringBoot 将本类中所有的属性和配置文件中相关的配置进行绑定
 * prefix = "person": 配置文件中哪个下面的所有属性进行映射
 *
 * 只有这个组件是容器中的组件才能使用容器提供的功能
 */
@Component
@ConfigurationProperties(prefix = "person")
public class Person {

    private String name;
    private Integer age;
    private Boolean boss;
    private Date birth;

    private Map<String, Object> maps;
    private List<Object> lists;
    private Dog dog;
}
```

`@ConfigurationProperties(prefix = "person")` 表示从配置文件中获取前缀为 "person" 的属性值。

获取配置之前可以导入配置文件处理器，以后编写配置就有提示：

```xml
<!--导入配置文件处理器，配置文件进行绑定就会有提示-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-configuration-processor</artifactId>
    <optional>true</optional>
</dependency>
```

- **@Value 获取值和 @ConfigurationProperties 获取值的比较：** 

|                      |   @ConfigurationProperties    |      @Value       |
| :------------------: | :---------------------------: | :---------------: |
|         功能         |   批量注入配置文件中的属性    |   一个一个指定    |
| 松散绑定（松散语法） | 支持（lastName == last_name） |      不支持       |
|         SpEL         |            不支持             | 支持（ #{11*2} ） |
|    JSR303数据校验    |             支持              |      不支持       |
|     复杂类型封装     |             支持              |      不支持       |

- **配置文件注入值数据校验：** 

```java
@Component
@ConfigurationProperties(prefix = "person")
@Validated
public class Person {

    @Email
    private String name;
    private Integer age;
    private Boolean boss;
    private Date birth;

    private Map<String, Object> maps;
    private List<Object> lists;
    private Dog dog;
}
```

> @ConfigurationProperties 和 @Value 的选择：
>
> 在某个业务逻辑中需要获取配置文件中的某项值，使用 @Value；
>
> 如果 JavaBean 需要和配置文件进行映射获取值，使用 @ConfigurationProperties

- **@PropertySource 和 @ImportResource：** 

默认情况下程序会加载默认的配置文件 application.yml 文件，注入里面的对应属性值。如果 application.yml 文件和指定加载的配置文件中都有相同的 person 开头的属性设置则会优先加载 application.yml 中的属性配置，和Spring Boot 的加载配置文件的优先级。

**@PropertySource：** 加载指定的配置文件：

```java
@Component
//加载类路径下的 "person.properties" 文件并注入属性值
@PropertySource(value={"classpath:person.properties"})
@ConfigurationProperties(prefix = "person")
@Validated
public class Person {

    @Email
    private String name;
    private Integer age;
    private Boolean boss;
    private Date birth;

    private Map<String, Object> maps;
    private List<Object> lists;
    private Dog dog;
}
```

**@ImportResource：** 导入 Spring 的配置文件，让配置文件中的内容生效

Spring Boot 中没有 Spring 的配置文件，自己编写的配置文件 Spring 也不能自动识别；若要加载 Spring 的配置文件就需要将 @ImportResource 标注在一个配置类上。

```java
//导入自定义的Spring配置文件，让其生效
@ImportResource(locations = {"classpath:beans.xml"})
@SpringBootApplication
public class SpringBoot02ConfigApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringBoot02ConfigApplication.class, args);
    }

}
```

**Spring Boot 推荐给容器中添加组件的方式：** 推荐使用权注解的方式

原始方式：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="helloService" class="com.yunhui.springboot.service.HelloService"></bean>
</beans>
```

1、编写配置类（类似Spring的配置文件）

2、使用 @Bean 给容器中添加组件

```java
/**
 * @Configuration 指明当前类是一个配置类，相当于之前的 Spring 配置文件
 *
 * 在配置文件中使用 <bean></bean> 标签添加组件
 */
@Configuration
public class MyAppConfig {
    //将方法的返回值添加到容器中，容器中组件默认的id就是方法名
    @Bean
    public HelloService helloService(){
        return new HelloService();
    }
}
```

#### 4、配置文件占位符：

随机数：

\${random.value}，\${random.int}，\${random.long}，\${random.int(10)}，\${random.int[1024,65535]}

占位符获取之前配置的值，如果没有值获取可以使用 `:` 指定默认值

```properties
person.name=lisi	
person.age=20
person.boss=false
person.cat.name=${person.name:hello}_cat
```

#### 5、Profile：

- **多 Profile 文件：** 

    在主配置文件编写时，文件名可以是 `application-{profile}.peoperties/yml` 默认使用 `application.properties` 文件的配置。

    application.properties

    application-dev.properties

    application-prod.properties

- **yml 支持多文档块方式：** 

    ```yaml
    server:
      port: 8081
    
    spring:
      profiles:
        active: dev
    ---
    
    server:
      port: 8083
    spring:
      profiles: dev
    
    ---
    
    server:
      port: 8084
    spring:
      profiles: prod	#指定属于哪个文档块
    ```

    yml 支持使用 `---` 三个短横线来划分文档块。通过 `spring.profiles` 来指定文档块的名称，使用 `spring.profiles.active` 来激活指定的文档块。

- **激活指定的 profile ：** 

    1）、在配置文件中通过 `spring.profiles.active=dev` 来激活指定的配置文件。

    ```properties
    spring.profiles.active=dev
    ```

    2）、命令行激活： `--spring.profiles.active=dev` 

    ```cmd
    java -jar jar包名 --spring.profiles.actice=dev
    ```

    3）、虚拟机参数： `-Dspring.profiles.active=dev`

#### 6、配置文件加载位置：

Spring Boot 启动会扫描以下位置的 application.properties 或者 application.yml 文件作为 Spring Boot 的默认配置文件。

> file:./config
>
> file:./
>
> classpath:/config/
>
> classpath:/
>
> **以上文件位置优先级由高到低，高优先级的配置会覆盖低优先级的配置** 

SpringBoot 会从这四个位置全部加载主配置文件，会形成互补配置。

通过 `spring.config.location` 来改变默认的配置文件位置 ，该配置项写在上述4个位置的配置文件中将不会起作用，可以使用命令行参数的形式，在项目启动的时候来指定配置文件的位置，指定的配置文件和默认加载的配置文件共同起作用，形成互补配置。

```cmd
java -jar jar包名 --spring.config.location=G:/application.properties
```

#### 7、外部配置加载顺序：

优先级由高到低：

**1）、命令行参数**

所有的配置都可以在命令行上进行指定

java -jar spring-boot-02-config-02-0.0.1-SNAPSHOT.jar --server.port=8087  --server.context-path=/abc

多个配置用空格分开； --配置项=值

2）、来自java:comp/env的JNDI属性

3）、Java系统属性（System.getProperties()）

4）、操作系统环境变量

5）、RandomValuePropertySource配置的random.*属性值

==**由jar包外向jar包内进行寻找；**==

==**优先加载带profile**==

**6）、jar包外部的application-{profile}.properties或application.yml(带spring.profile)配置文件** 

**7）、jar包内部的application-{profile}.properties或application.yml(带spring.profile)配置文件** 

==**再来加载不带profile**==

**8）、jar包外部的application.properties或application.yml(不带spring.profile)配置文件** 

**9）、jar包内部的application.properties或application.yml(不带spring.profile)配置文件** 

10）、@Configuration注解类上的@PropertySource

11）、通过SpringApplication.setDefaultProperties指定的默认属性

所有支持的配置加载来源；[参考官方文档](https://docs.spring.io/spring-boot/docs/1.5.9.RELEASE/reference/htmlsingle/#boot-features-external-config)

#### 8、自动配置原理：

配置文件里能配置的属性参照：https://docs.spring.io/spring-boot/docs/1.5.9.RELEASE/reference/htmlsingle/#common-application-properties

- **自动配置原理：** 

    1）、Spring Boot 启动的时候加载主配置类，并开启了自动配置功能 `@EnableAutoConfiguration` 

    2）、`@EnableAutoConfiguration` 作用：利用 `@Import(EnableAutoConfigurationImportSelector.class)` 给容器中导入一些组件。可以查看 `selectImports(AnnotationMetadata annotationMetadata)` 方法内容：

    ```java
    //获取候选的配置
    List<String> configurations = this.getCandidateConfigurations(annotationMetadata, attributes);
    
    //扫描所有jar包类路径下的 META-INF/spring.factories 文件，把扫描到的文件内容包装成 properties 对象
    SpringFactoriesLoader.loadFactoryNames(this.getSpringFactoriesLoaderFactoryClass(), this.getBeanClassLoader());
    
    //从 properties 中获取到 EnableAutoConfiguration.class 类（类名）的值，再添加到容器中
    ```

    > 将每个jar包类路径下 META-INF/spring.factories 里配置的多有 EnableAutoConfiguration 的值加入到容器中。
    >
    > 每一个 xxxAutoConfiguration 类都是容器中的一个组件，都加入到容器中，用它们来做自动配置。

    3）、每一个自动配置类进行自动配置功能

    4）、以 `HttpEncodingAutoConfiguration` （Http编码自动配置）为例解释自动配置原理：

    ```java
    //表示这是一个配置类，类似于配置文件，也可以给容器中添加组件
    @Configuration
    
    //启用指定类的 @ConfigurationProperties 功能；将配置文件中对应的值和指定类（HttpEncodingProperties.class）的属性进行绑定，并把 HttpEncodingProperties 加入到 IOC 容器中
    @EnableConfigurationProperties(HttpEncodingProperties.class)
    
    //Spring底层 @Conditional 注解，根据不同的条件，如果满足指定的条件，整个配置类里面的配置就会生效，该注解判断当前应用是否是 WEB 应用，如果是，当前配置类生效，如果不是则不生效
    @ConditionalOnWebApplication
    
    //判断当前项目有没有 CharacterEncodingFilter.class 这个类。CharacterEncodingFilter 是SpringMVC中解决乱码问题的过滤器
    @ConditionalOnClass(CharacterEncodingFilter.class)
    
    //判断配置文件中是否存在某个配置（spring.http.encoding.enabled），如果不存在判断也是成立的；即使配置文件中不配置 spring.http.encoding.enabled=true 也是默认生效的
    @ConditionalOnProperty(prefix = "spring.http.encoding", value = "enabled", matchIfMissing = true)
    public class HttpEncodingAutoConfiguration {
    
        //已经和 SpringBoot 的配置文件映射
    	private final HttpEncodingProperties properties;
    
        //只有一个有参构造器的情况下，参数的值就会从容器中拿
    	public HttpEncodingAutoConfiguration(HttpEncodingProperties properties) {
    		this.properties = properties;
    	}
    
    	@Bean //给容器中添加组件，这个组件的某些值需要从 properties 中获取
    	@ConditionalOnMissingBean(CharacterEncodingFilter.class) //判断容器中没有这个组件
    	public CharacterEncodingFilter characterEncodingFilter() {
    		CharacterEncodingFilter filter = new OrderedCharacterEncodingFilter();
    		filter.setEncoding(this.properties.getCharset().name());
    		filter.setForceRequestEncoding(this.properties.shouldForce(Type.REQUEST));
    		filter.setForceResponseEncoding(this.properties.shouldForce(Type.RESPONSE));
    		return filter;
    	}
    }
    ```

    > 自动配置类根据当前不同的条件进行判断，决定这个配置是否生效；一旦这个配置类生效，配置类就会向容器中添加组件；这些组件的属性是从对应的 properties 类中获取的，这些类里面的每一个属性又是和配置文件绑定的。

    5）、所有在配置文件中能配置的属性都是在 xxxProperties 类中封装着；配置文件能配置什么就可以参照某一个功能对应的属性类

    ```java
    @ConfigurationProperties(prefix = "spring.http.encoding")//从配置文件中获取指定的值和bean的属性进行绑定
    public class HttpEncodingProperties {
    	public static final Charset DEFAULT_CHARSET = Charset.forName("UTF-8");
    }
    ```

    > 1、SpringBoot 启动会加载大量的自动配置类
    >
    > 2、查看我们需要的功能有没有 SpringBoot 默认写好的自动配置类
    >
    > 3、如果有，再来看这个自动配置类中配置了哪些组件（有需要的组件就不需要再来配置，没有就需要自己写配置类）
    >
    > 4、自动配置类给容器中添加组件的时候，会从 properties 类中获取某些属性，这些属性就可以在配置文件中指定值

- **细节：** 

    1）、**@Conditional 派生注解：** 

    作用： 必须是 @Conditional 指定的条件成立，才给容器中添加组件，配置里的所有内容才会生效

    | @Conditional扩展注解            | 作用（判断是否满足当前指定条件）                 |
    | ------------------------------- | ------------------------------------------------ |
    | @ConditionalOnJava              | 系统的java版本是否符合要求                       |
    | @ConditionalOnBean              | 容器中存在指定Bean；                             |
    | @ConditionalOnMissingBean       | 容器中不存在指定Bean；                           |
    | @ConditionalOnExpression        | 满足SpEL表达式指定                               |
    | @ConditionalOnClass             | 系统中有指定的类                                 |
    | @ConditionalOnMissingClass      | 系统中没有指定的类                               |
    | @ConditionalOnSingleCandidate   | 容器中只有一个指定的Bean，或者这个Bean是首选Bean |
    | @ConditionalOnProperty          | 系统中指定的属性是否有指定的值                   |
    | @ConditionalOnResource          | 类路径下是否存在指定资源文件                     |
    | @ConditionalOnWebApplication    | 当前是web环境                                    |
    | @ConditionalOnNotWebApplication | 当前不是web环境                                  |
    | @ConditionalOnJndi              | JNDI存在指定项                                   |

    **自动配置类必须在一定的条件下才能生效：** 

    可以通过在配置文件中启用 debug=true 属性，让控制台打印自动配置报告：

    ```xml
    =========================
    AUTO-CONFIGURATION REPORT
    =========================
    
    
    Positive matches:(启用的自动配置类)
    -----------------
    
       DispatcherServletAutoConfiguration matched:
          - @ConditionalOnClass found required class 'org.springframework.web.servlet.DispatcherServlet'; @ConditionalOnMissingClass did not find unwanted class (OnClassCondition)
          - @ConditionalOnWebApplication (required) found StandardServletEnvironment (OnWebApplicationCondition)
    
       DispatcherServletAutoConfiguration.DispatcherServletConfiguration matched:
          - @ConditionalOnClass found required class 'javax.servlet.ServletRegistration'; @ConditionalOnMissingClass did not find unwanted class (OnClassCondition)
          - Default DispatcherServlet did not find dispatcher servlet beans (DispatcherServletAutoConfiguration.DefaultDispatcherServletCondition)
    
    Negative matches:（没有启用的自动配置类）
    -----------------
    ```

## 三、日志

#### 1、日志框架：

JUL、JCL、Jboss-logging、logback、log4j、log4j2、slf4j....

| 日志门面  （日志的抽象层）                                   | 日志实现                                             |
| ------------------------------------------------------------ | ---------------------------------------------------- |
| ~~JCL（Jakarta  Commons Logging）~~    SLF4j（Simple  Logging Facade for Java）    ~~jboss-logging~~ | Log4j  JUL（java.util.logging）  Log4j2  **Logback** |

日志门面：SLF4j                                    日志实现：Logback

SpringBoot：底层是 Spring 框架，Spring 默认使用 JCL ；但 SpringBoot 选用 SLF4j 和 Logback

#### 2、SLF4j 使用：

**1）、如何在系统中使用 SLF4j ：** 以后开发的时候，日志记录方法的调用，不应该直接调用日志的实现类，而是直接调用日志抽象层里的方法。

首先给系统中导入 SLF4j 和 Logback 的 jar 包

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class HelloWorld {
  public static void main(String[] args) {
    Logger logger = LoggerFactory.getLogger(HelloWorld.class);
    logger.info("Hello World");
  }
}
```

![SLF4J的使用](G:\云汇科技\笔记\浙思云汇\assets/concrete-bindings.png)

每一个日志的实现都有自己的配置文件。使用 SLF4j 后，配置文件还是做成日志实现框架的配置文件。

**2）、统一日志记录：** 即使是别的日志框架也统一使用 SLF4j 进行输出

![](G:\云汇科技\笔记\浙思云汇\assets\legacy.png)

> 如何将系统中所有的日志都统一到 SLF4j ：
>
> 1、将系统中其它日志框架先排除出去；
>
> 2、用中间包来替换原有的日志框架；
>
> 3、再导入 SLF4j 的其它实现。

#### 3、SpringBoot 日志关系：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter</artifactId>
</dependency>
```

SpringBoot 使用 spring-boot-starter-logging 做日志功能

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-logging</artifactId>
    <version>2.1.1.RELEASE</version>
    <scope>compile</scope>
</dependency>
```

SpringBoot 底层依赖关系：

![](G:\云汇科技\笔记\浙思云汇\assets\20180131220946.png)

> 1、SpringBoot 底层也是使用 SLF4j + Logback 的方式进行日志记录
>
> 2、SpringBoot 也把其它的日志都替换成了 SLF4j 
>
> 3、替换中间转换包

在 SpringBoot 中使用其它框架的时候一定要移除框架的日志包：

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-core</artifactId>
    <exclusions>
        <exclusion>
            <groupId>commons-logging</groupId>
            <artifactId>commons-logging</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

#### 4、日志使用：

**1）、默认配置：** 

SpringBoot 默认已经配置好了日志：

```java
//日志记录器
private Logger logger = LoggerFactory.getLogger(getClass());

@Test
public void contextLoads() {
    //日志的级别，由低到高 trace < debug < info < warn < error
    //可以调整输出的日志级别，日志就只会在该级别以及更高级别生效
    logger.trace("trace日志");
    logger.debug("debug日志");
    //SpringBoot 默认日志级别是 info, 没有指定级别的就使用 SpringBoot 默认的级别 root 级别
    logger.info("info日志");
    logger.warn("warn日志");
    logger.error("error日志");
}
```

日志格式：

```xml
<!--
    日志输出格式：
        %d表示日期时间，
        %thread表示线程名，
        %-5level：级别从左显示5个字符宽度
        %logger{50} 表示logger名字最长50个字符，否则按照句点分割。 
        %msg：日志消息，
        %n是换行符
-->
%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n
```

Spring Boot 修改日志的默认配置：

```properties
logging.level.com.yunhui=trace

# 不指定路径在当前项目下生成 spring.log 日志文件
# 可以指定完整的路径
logging.file=spring.log

# 在当前磁盘的根路径下创建 spring 文件夹和 log 文件夹；使用 spring.log 作为默认文件
logging.path=/spring/log

# 在控制台输出的日志格式
logging.pattern.console=%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n
# 指定文件中日志输出的格式
logging.pattern.file=%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n
```

| logging.file | logging.path | Example  | Description                        |
| ------------ | ------------ | -------- | ---------------------------------- |
| (none)       | (none)       |          | 只在控制台输出                     |
| 指定文件名   | (none)       | my.log   | 输出日志到my.log文件               |
| (none)       | 指定目录     | /var/log | 输出到指定目录的 spring.log 文件中 |

**2）、指定配置：** 

在类路径下放上每个日志框架自己的配置文件，SpringBoot 就会自动使用类路径下的日志文件

| Logging System          | Customization                                                |
| ----------------------- | ------------------------------------------------------------ |
| Logback                 | `logback-spring.xml`, `logback-spring.groovy`, `logback.xml` or `logback.groovy` |
| Log4j2                  | `log4j2-spring.xml` or `log4j2.xml`                          |
| JDK (Java Util Logging) | `logging.properties`                                         |

如果使用 logback.xml 会直接被日志框架识别，就绕过了 SpringBoot ；官方推荐使用 logback-spring.xml 命名方式。logback-spring.xml 日志框架就不直接加载日志的配置，由 SpringBoot 加载解析，并可以使用 \<springProfile> 标签。

```xml
<springProfile name="staging">
    <!-- configuration to be enabled when the "staging" profile is active -->
  	可以指定某段配置只在某个环境下生效
</springProfile>
```

```xml
<layout class="ch.qos.logback.classic.PatternLayout">
    <springProfile name="dev">
        <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} --> [%thread] %-5level %logger{50} - %msg%n</pattern>
    </springProfile>

    <springProfile name="!dev">
        <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} === [%thread] %-5level %logger{50} - %msg%n</pattern>
    </springProfile>
</layout>

<!--
	使用 <springProfile name="dev"> 标签可以指定在不同环境下使用不同形式的日志格式输出
-->
```

#### 5、切换日志框架：

1）、排除掉其它日志包

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-web</artifactId>
  <exclusions>
    <exclusion>
      <artifactId>logback-classic</artifactId>
      <groupId>ch.qos.logback</groupId>
    </exclusion>
    <exclusion>
      <artifactId>log4j-over-slf4j</artifactId>
      <groupId>org.slf4j</groupId>
    </exclusion>
  </exclusions>
</dependency>

<dependency>
  <groupId>org.slf4j</groupId>
  <artifactId>slf4j-log4j12</artifactId>
</dependency>
```

2）、切换为 log4j2： 排除 spring-boot-starter-logging ，引入 spring-boot-starter-log4j2 即可完成切换

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
	<exclusions>
        <exclusion>
            <artifactId>spring-boot-starter-logging</artifactId>
            <groupId>org.springframework.boot</groupId>
        </exclusion>
	</exclusions>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-log4j2</artifactId>
</dependency>
```

## 四、Web 开发

#### 1、简介：

使用 SpringBoot ：

- 创建 SpringBoot 应用，选中需要的模块
- SpringBoot 默认已经配置好选中模块的使用场景，只需要在配置文件中指定少量的配置即可运行起来
- 编写业务代码

自动配置原理：这个场景 SpringBoot 帮我们配置了什么，能不能修改，能修改哪些，能不能拓展。

```xml
xxxxAutoConfiguration：帮我们给容器中自动配置组件；
xxxxProperties:配置类来封装配置文件的内容；
```

#### 2、Spring Boot 对静态资源的映射规则：

```java
@ConfigurationProperties(prefix = "spring.resources", ignoreUnknownFields = false)
public class ResourceProperties {}
//可以设置和静态资源有关的参数，如缓存时间等
```

```java
@Override
public void addResourceHandlers(ResourceHandlerRegistry registry) {
    if (!this.resourceProperties.isAddMappings()) {
        logger.debug("Default resource handling disabled");
        return;
    }
    Duration cachePeriod = this.resourceProperties.getCache().getPeriod();
    CacheControl cacheControl = this.resourceProperties.getCache()
            .getCachecontrol().toHttpCacheControl();
    if (!registry.hasMappingForPattern("/webjars/**")) {
        customizeResourceHandlerRegistration(registry
                .addResourceHandler("/webjars/**")
                .addResourceLocations("classpath:/META-INF/resources/webjars/")
                .setCachePeriod(getSeconds(cachePeriod))
                .setCacheControl(cacheControl));
    }
    String staticPathPattern = this.mvcProperties.getStaticPathPattern();
    if (!registry.hasMappingForPattern(staticPathPattern)) {
        customizeResourceHandlerRegistration(
                registry.addResourceHandler(staticPathPattern)
                        .addResourceLocations(getResourceLocations(
                                this.resourceProperties.getStaticLocations()))
                        .setCachePeriod(getSeconds(cachePeriod))
                        .setCacheControl(cacheControl));
    }
}

//配置欢迎页面映射
@Bean
public WelcomePageHandlerMapping welcomePageHandlerMapping(
        ApplicationContext applicationContext) {
    return new WelcomePageHandlerMapping(
            new TemplateAvailabilityProviders(applicationContext),
            applicationContext, getWelcomePage(),
            this.mvcProperties.getStaticPathPattern());
}

//配置网页标签页的小图标
@Configuration
@ConditionalOnProperty(value = "spring.mvc.favicon.enabled", matchIfMissing = true)
public static class FaviconConfiguration implements ResourceLoaderAware {

    private final ResourceProperties resourceProperties;

    private ResourceLoader resourceLoader;

    public FaviconConfiguration(ResourceProperties resourceProperties) {
        this.resourceProperties = resourceProperties;
    }

    @Override
    public void setResourceLoader(ResourceLoader resourceLoader) {
        this.resourceLoader = resourceLoader;
    }

    @Bean
    public SimpleUrlHandlerMapping faviconHandlerMapping() {
        SimpleUrlHandlerMapping mapping = new SimpleUrlHandlerMapping();
        mapping.setOrder(Ordered.HIGHEST_PRECEDENCE + 1);
        mapping.setUrlMap(Collections.singletonMap("**/favicon.ico",
                faviconRequestHandler()));
        return mapping;
    }

    @Bean
    public ResourceHttpRequestHandler faviconRequestHandler() {
        ResourceHttpRequestHandler requestHandler = new ResourceHttpRequestHandler();
        requestHandler.setLocations(resolveFaviconLocations());
        return requestHandler;
    }

    private List<Resource> resolveFaviconLocations() {
        String[] staticLocations = getResourceLocations(
                this.resourceProperties.getStaticLocations());
        List<Resource> locations = new ArrayList<>(staticLocations.length + 1);
        Arrays.stream(staticLocations).map(this.resourceLoader::getResource)
                .forEach(locations::add);
        locations.add(new ClassPathResource("/"));
        return Collections.unmodifiableList(locations);
    }

}
```

**1）、映射规则一：** 所有 `webjars/**` 的请求都去 `classpath:/META-INF/resources/webjars/` 查找资源；

webjars： 以 jar 包的方式引入静态资源 http://www.webjars.org/

例如： 引入 jquery 文件：

```xml
<!--引入 jquery-webjars-->
<dependency>
    <groupId>org.webjars</groupId>
    <artifactId>jquery</artifactId>
    <version>3.3.1</version>
</dependency>
```

此时在浏览器中就可以通过 localhost:8080/webjars/jquery/3.3.1/jquery.js 访问该资源

**2）、映射规则二：** `/**` 访问当前项目的任何资源

```java
//获取到静态资源的路径，由 WebMvcProperties 中的 staticPathPattern 属性提供，默认为 /**
String staticPathPattern = this.mvcProperties.getStaticPathPattern();
if (!registry.hasMappingForPattern(staticPathPattern)) {
    customizeResourceHandlerRegistration(
            registry.addResourceHandler(staticPathPattern)
                    .addResourceLocations(getResourceLocations(
                            this.resourceProperties.getStaticLocations()))
                    .setCachePeriod(getSeconds(cachePeriod))
                    .setCacheControl(cacheControl));
}
```

如果没有程序处理这些请求则会通过 `this.resourceProperties.getStaticLocations()` 去以下路径查找：

静态资源文件夹：

```java
"classpath:/META-INF/resources/", 
"classpath:/resources/",
"classpath:/static/", 
"classpath:/public/",
"/" //当前项目的根路径
```

**3）、配置欢迎页面：** 静态资源文件夹下的所有 index.html 页面，被 `/**` 映射。

访问 localhost:8080/ 默认会去查找静态资源路径下的 index.html 页面。

**4）、配置小图标：** 所有的 `**/favicon.ico` 都在静态资源文件夹下找

#### 3、模板引擎：

JSP、Velocity、Freemarker、Thymeleaf

![](G:\云汇科技\笔记\浙思云汇\assets\template-engine.png)

**SpringBoot 推荐使用的 Thymeleaf 模板引擎：** 语法更简单，功能更强大

##### 1）、引入 thymeleaf：

```xml
<!--thymeleaf-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>

切换thymeleaf版本
<properties>
	<thymeleaf.version>3.0.9.RELEASE</thymeleaf.version>
	<!-- 布局功能的支持程序  thymeleaf3主程序  layout2以上版本 -->
	<!-- thymeleaf2   layout1-->
	<thymeleaf-layout-dialect.version>2.2.2</thymeleaf-layout-dialect.version>
</properties>
```

##### 2）、Thymeleaf 使用和语法：

```java
@ConfigurationProperties(prefix = "spring.thymeleaf")
public class ThymeleafProperties {

	private static final Charset DEFAULT_ENCODING = StandardCharsets.UTF_8;

	public static final String DEFAULT_PREFIX = "classpath:/templates/";

	public static final String DEFAULT_SUFFIX = ".html";
}
```

> 只要把 html 页面放在 classpath:/templates/ 文件夹中 tymeleaf 就能自动渲染

**使用步骤：** 

1）、导入 thymeleaf 的名称空间：

```html
<html lang="en" xmlns:th="http://www.thymeleaf.org">
```

2）、thymeleaf 语法：

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Success</title>
</head>
<body>
    <h2>成功</h2>
    <!-- th:text="" 将div中的文本内容设置为指定的值-->
    <div th:text="${hello}"></div>
</body>
</html>
```

##### 3）、语法规则：

- **th:text=" "：** 改变当前元素里的文本内容。可以使用 `th:任意html属性` 来替换原来的属性。

    ![](G:\云汇科技\笔记\浙思云汇\assets\2018-02-04_123955.png)

- **表达式：** 

    ```properties
    Simple expressions:（表达式语法）
        Variable Expressions: ${...}：获取变量值；OGNL；
        		1）、获取对象的属性、调用方法
        		2）、使用内置的基本对象：
        			#ctx : the context object. 上下文对象
        			#vars: the context variables.
                    #locale : the context locale.
                    #request : (only in Web Contexts) the HttpServletRequest object.
                    #response : (only in Web Contexts) the HttpServletResponse object.
                    #session : (only in Web Contexts) the HttpSession object.
                    #servletContext : (only in Web Contexts) the ServletContext object.
                    
                    ${session.foo}
                3）、内置的一些工具对象：
                #execInfo : information about the template being processed.
                #messages : methods for obtaining externalized messages inside variables expressions, in the same way as they would be obtained using #{…} syntax.
                #uris : methods for escaping parts of URLs/URIs
                #conversions : methods for executing the configured conversion service (if any).
                #dates : methods for java.util.Date objects: formatting, component extraction, etc.
                #calendars : analogous to #dates , but for java.util.Calendar objects.
                #numbers : methods for formatting numeric objects.
                #strings : methods for String objects: contains, startsWith, prepending/appending, etc.
                #objects : methods for objects in general.
                #bools : methods for boolean evaluation.
                #arrays : methods for arrays.
                #lists : methods for lists.
                #sets : methods for sets.
                #maps : methods for maps.
                #aggregates : methods for creating aggregates on arrays or collections.
                #ids : methods for dealing with id attributes that might be repeated (for example, as a result of an iteration).
    
        Selection Variable Expressions: *{...}：选择表达式：和${}在功能上是一样；
        	补充：配合 th:object="${session.user}：
            <div th:object="${session.user}">
                <p>Name: <span th:text="*{firstName}">Sebastian</span>.</p>
                <p>Surname: <span th:text="*{lastName}">Pepper</span>.</p>
                <p>Nationality: <span th:text="*{nationality}">Saturn</span>.</p>
            </div>
        
        Message Expressions: #{...}：获取国际化内容
        Link URL Expressions: @{...}：定义URL；
        		@{/order/process(execId=${execId},execType='FAST')}
        Fragment Expressions: ~{...}：片段引用表达式
        		<div th:insert="~{commons :: main}">...</div>
        		
    Literals（字面量）
          Text literals: 'one text' , 'Another one!' ,…
          Number literals: 0 , 34 , 3.0 , 12.3 ,…
          Boolean literals: true , false
          Null literal: null
          Literal tokens: one , sometext , main ,…
          
    Text operations:（文本操作）
        String concatenation: +
        Literal substitutions: |The name is ${name}|
        
    Arithmetic operations:（数学运算）
        Binary operators: + , - , * , / , %
        Minus sign (unary operator): -
        
    Boolean operations:（布尔运算）
        Binary operators: and , or
        Boolean negation (unary operator): ! , not
        
    Comparisons and equality:（比较运算）
        Comparators: > , < , >= , <= ( gt , lt , ge , le )
        Equality operators: == , != ( eq , ne )
        
    Conditional operators:条件运算（三元运算符）
        If-then: (if) ? (then)
        If-then-else: (if) ? (then) : (else)
        Default: (value) ?: (defaultvalue)
        
    Special tokens:
        No-Operation: _ 
    ```

#### 4、SpringMVC 自动配置：

https://docs.spring.io/spring-boot/docs/1.5.10.RELEASE/reference/htmlsingle/#boot-features-developing-web-applications

##### 1）、Spring MVC auto-configuration：

SpringBoot 已经自动配置好了 SpringMVC，以下是 SpringBoot 对 SpringMVC 的默认配置：（`WebMvcAutoConfiguration`）

- Inclusion of `ContentNegotiatingViewResolver` and `BeanNameViewResolver` beans.

    - 自动配置了ViewResolver（视图解析器：根据方法的返回值得到视图对象（View），视图对象决定如何渲染（转发 / 重定向））
    - `ContentNegotiatingViewResolver` 组合所有的视图解析器
    - 如何定制：我们可以自己给容器中添加一个视图解析器；自动的将其组合进来

- Support for serving static resources, including support for WebJars (see below).     静态资源文件夹路径,webjars

- Static `index.html` support. 静态首页访问

- Custom `Favicon` support (see below).  favicon.ico

- 自动注册了 of `Converter`, `GenericConverter`, `Formatter` beans.

    - `Converter`  ： 转换器；  public String hello(User user)：类型转换使用Converter

    - `Formatter`  ： 格式化器；  2017.12.17===Date；

        ==自己添加的格式化器转换器，我们只需要放在容器中即可== 

- Support for `HttpMessageConverters` (see below).

    - `HttpMessageConverter` ：SpringMVC用来转换Http请求和响应的；User ---> Json；

    - `HttpMessageConverters` 是从容器中确定；获取所有的 HttpMessageConverter；

        ==自己给容器中添加HttpMessageConverter，只需要将自己的组件注册容器中（@Bean，@Component）==​

        

- Automatic registration of `MessageCodesResolver` (see below).定义错误代码生成规则

- Automatic use of a `ConfigurableWebBindingInitializer` bean (see below). 

    ==我们可以配置一个 `ConfigurableWebBindingInitializer` 来替换默认的；（添加到容器）==

    ```java
    初始化WebDataBinder；
    请求数据 ----> JavaBean；
    ```

    **org.springframework.boot.autoconfigure.web：web所有的自动配置场景** 

##### 2）、扩展SpringMVC：

```xml
<mvc:view-controller path="/hello" view-name="success"/>
<mvc:interceptors>
    <mvc:interceptor>
        <mvc:mapping path="/hello"/>
        <bean></bean>
    </mvc:interceptor>
</mvc:interceptors>
```

**编写一个配置类（@Configuration），是WebMvcConfigurerAdapter类型；不能标注@EnableWebMvc** 

扩展 SpringMVC：既保留了所有自动配置，也能用我们扩展的配置

```java
//使用WebMvcConfigurerAdapter可以来扩展SpringMVC的功能
@Configuration
public class MyMvcConfig extends WebMvcConfigurerAdapter {

    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        //super.addViewControllers(registry);
        //浏览器发送 /abc 请求来到 success.html 页面
        registry.addViewController("/abc").setViewName("success");
    }
}
```

原理：

​	1）、`WebMvcAutoConfiguration` 是 SpringMVC 的自动配置类

​	2）、在做其他自动配置时会导入：@Import(**EnableWebMvcConfiguration**.class)

```java
@Configuration
public static class EnableWebMvcConfiguration extends DelegatingWebMvcConfiguration {
  private final WebMvcConfigurerComposite configurers = new WebMvcConfigurerComposite();

 //从容器中获取所有的WebMvcConfigurer
  @Autowired(required = false)
  public void setConfigurers(List<WebMvcConfigurer> configurers) {
      if (!CollectionUtils.isEmpty(configurers)) {
          this.configurers.addWebMvcConfigurers(configurers);
            //一个参考实现；将所有的 WebMvcConfigurer 相关配置都来一起调用；  
            @Override
         // public void addViewControllers(ViewControllerRegistry registry) {
         //     for (WebMvcConfigurer delegate : this.delegates) {
         //         delegate.addViewControllers(registry);
         //     }
          }
      }
}
```

​	3）、容器中所有的 WebMvcConfigurer 都会一起起作用

​	4）、自己的配置类也会被调用，效果：SpringMVC 的自动配置和我们的扩展都会起作用

##### 3）、全面接管 SpringMVC：

使用 **@EnableWebMvc** 注解会抛弃 SpringBoot 对 SpringMVC 的所有配置，所有的配置都需要自己写。所有的SpringMVC 的自动配置都失效。

原理：

为什么使用 `@EnableWebMvc` 注解后 SpringMVC 的自动配置就失效了：

1）@EnableWebMvc的核心

```java
@Import(DelegatingWebMvcConfiguration.class)
public @interface EnableWebMvc {
```

2）、

```java
@Configuration
public class DelegatingWebMvcConfiguration extends WebMvcConfigurationSupport {
```

3）、`WebMvcAutoConfiguration` 类的头部注解

```java
@Configuration
@ConditionalOnWebApplication
@ConditionalOnClass({ Servlet.class, DispatcherServlet.class,
		WebMvcConfigurerAdapter.class })
//容器中没有这个组件的时候，这个自动配置类才生效
@ConditionalOnMissingBean(WebMvcConfigurationSupport.class)
@AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE + 10)
@AutoConfigureAfter({ DispatcherServletAutoConfiguration.class,
		ValidationAutoConfiguration.class })
public class WebMvcAutoConfiguration {
```

4）、@EnableWebMvc 将 WebMvcConfigurationSupport 组件导入进来；

5）、导入的 WebMvcConfigurationSupport 只是 SpringMVC 最基本的功能；

#### 5、如何修改 SpringBoot 的默认配置：

模式：

​	1）、SpringBoot在自动配置很多组件的时候，先看容器中有没有用户自己配置的（@Bean、@Component）如果有就使用用户配置的，如果没有，才自动配置；如果有些组件可以有多个（ViewResolver）将用户配置的和自己默认的组合起来；

​	2）、在SpringBoot中会有非常多的 xxxConfigurer 帮助我们进行 **扩展配置** 

​	3）、在SpringBoot中会有很多的 xxxCustomizer 帮助我们进行 **定制配置** 

#### 6、RestfulCRUD案例：

1）、默认访问首页：

```java
//在 controller 中写一个方法，映射地址为 "/" 和 "index.html"
@RequestMapping({"/", "index.html"})
public String index(){
    return "index";
}
```

```java
//实现 WebMvcConfigurer 可以来扩展SpringMVC的功能
@Configuration
public class MyConfig implements WebMvcConfigurer {

    //将自定义组件注册在容器中
    @Bean
    public WebMvcConfigurer webMvcConfigurer(){
        return new WebMvcConfigurer() {
            @Override
            public void addViewControllers(ViewControllerRegistry registry) {
                registry.addViewController("/").setViewName("index");
                registry.addViewController("/index.html").setViewName("index");
            }
        };
    }
}
```

2）、国际化：

**SpringMVC国际化：** 

- 编写国际化配置文件：
- 使用 ResourceBundleMessageSource 管理国际化资源文件
- 在 JSP 页面使用 fmt:message 取出国际化内容

**SpringBoot 国际化：** 

- 编写国际化配置文件，抽取页面需要显示的国际化消息

    ```properties
    #中文
    login.password=密码
    login.remember=记住我
    login.btn=登录
    login.tip=请登录
    login.username=用户名
    ```

    ```properties
    #英文
    login.password=Password
    login.remember=Remember me
    login.btn=Sign in
    login.tip=Please sign in
    login.username=UserName
    ```

    ```properties
    #默认情况下
    login.password=密码
    login.remember=记住我
    login.btn=登录
    login.tip=请登录~
    login.username=用户名
    ```

- SpringBoot 已经配置好了自动管理国际化资源的组件

    ```java
    @Bean
    @ConfigurationProperties(prefix = "spring.messages")
    public MessageSourceProperties messageSourceProperties() {
        return new MessageSourceProperties();
    }
    
    @Bean
    public MessageSource messageSource(MessageSourceProperties properties) {
        ResourceBundleMessageSource messageSource = new ResourceBundleMessageSource();
        if (StringUtils.hasText(properties.getBasename())) {
            //设置国际化资源文件的基础名（去掉语言国家代码的）
            messageSource.setBasenames(StringUtils.commaDelimitedListToStringArray(
                    StringUtils.trimAllWhitespace(properties.getBasename())));
        }
        if (properties.getEncoding() != null) {
            messageSource.setDefaultEncoding(properties.getEncoding().name());
        }
        messageSource.setFallbackToSystemLocale(properties.isFallbackToSystemLocale());
        Duration cacheDuration = properties.getCacheDuration();
        if (cacheDuration != null) {
            messageSource.setCacheMillis(cacheDuration.toMillis());
        }
        messageSource.setAlwaysUseMessageFormat(properties.isAlwaysUseMessageFormat());
        messageSource.setUseCodeAsDefaultMessage(properties.isUseCodeAsDefaultMessage());
        return messageSource;
    }
    ```

- 去页面获取国际化的值：

    在页面使用 `#{}` 的形式进行取值

    ```html
    <body class="text-center">
        <form class="form-signin" action="dashboard.html">
            <img class="mb-4" src="../static/asserts/img/bootstrap-solid.svg" th:src="@{/asserts/img/bootstrap-solid.svg}" alt="" width="72" height="72">
            <h1 class="h3 mb-3 font-weight-normal" th:text="#{login.tip}">Please sign in</h1>
            <label class="sr-only" th:text="#{login.username}">Username</label>
            <input type="text" class="form-control" th:placeholder="#{login.username}" placeholder="Username" required="" autofocus="">
            <label class="sr-only" th:text="#{login.password}">Password</label>
            <input type="password" class="form-control" th:placeholder="#{login.password}" placeholder="Password" required="">
            <div class="checkbox mb-3">
                <label>
          <input type="checkbox" value="remember-me"> [[#{login.remember}]]
        </label>
            </div>
            <button class="btn btn-lg btn-primary btn-block" type="submit" th:text="#{login.btn}">Sign in</button>
            <p class="mt-5 mb-3 text-muted">© 2017-2018</p>
            <a class="btn btn-sm">中文</a>
            <a class="btn btn-sm">English</a>
        </form>
    </body>
    ```

    效果：根据浏览器语言信息切换国际化

    点击按钮切换国际化显示原理： 国际化 Locale（区域对象信息）；LocaleResolver（获取区域信息对象）

    ```java
    @Bean
    @ConditionalOnMissingBean
    @ConditionalOnProperty(prefix = "spring.mvc", name = "locale")
    public LocaleResolver localeResolver() {
        if (this.mvcProperties
                .getLocaleResolver() == WebMvcProperties.LocaleResolver.FIXED) {
            return new FixedLocaleResolver(this.mvcProperties.getLocale());
        }
        AcceptHeaderLocaleResolver localeResolver = new AcceptHeaderLocaleResolver();
        localeResolver.setDefaultLocale(this.mvcProperties.getLocale());
        return localeResolver;
    }
    //默认的区域信息配置是根据请求头带来的区域信息获取 Locale 进行国际化
    ```

    点击按钮切换国际化显示实现：手动编写一个 localeResolver 替换掉 SpringBoot 的默认配置。

    ```java
    /**
     * 可以在连接上携带区域信息
     */
    public class MyLocalResolver implements LocaleResolver {
    
        @Override
        public Locale resolveLocale(HttpServletRequest request) {
            String l = request.getParameter("l");
            if (!StringUtils.isEmpty(l)) {
                String[] s = l.split("_");
                return new Locale(s[0], s[1]);
            } else {
                return request.getLocale();
            }
        }
    
        @Override
        public void setLocale(HttpServletRequest request, HttpServletResponse response, Locale locale) {
    
        }
    }
    
    //编写好 LocaleResolver 后要将其添加至容器中
    @Bean
    public LocaleResolver localeResolver(){
        return new MyLocalResolver();
    }
    ```


3）、使用拦截器进行登录检查：

```java
/**
 * 设置登录检查的拦截器
 */
public class LoginHandlerInterceptor implements HandlerInterceptor {

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        Object user = request.getSession().getAttribute("user");
        if (user == null) {
            response.sendRedirect("/");
            return false;
        }
        return true;
    }
}
```

```java
//在组件中配置拦截器
@Bean
public WebMvcConfigurer webMvcConfigurer(){
    return new WebMvcConfigurer() {
        @Override
        public void addViewControllers(ViewControllerRegistry registry) {
            registry.addViewController("/").setViewName("index");
            registry.addViewController("/index.html").setViewName("index");
            registry.addViewController("/main.html").setViewName("dashboard");
        }

        //注册拦截器
        @Override
        public void addInterceptors(InterceptorRegistry registry) {
            //SpringBoot已经做好了静态资源映射，不用处理静态资源
            registry.addInterceptor(new LoginHandlerInterceptor()).addPathPatterns("/**")
            .excludePathPatterns("/index.html","/","/user/login");
        }
    };
}
```

4）、Restful风格：

|      | 普通CRUD（uri来区分操作） | RestfulCRUD       |
| ---- | ------------------------- | ----------------- |
| 查询 | getEmp                    | emp---GET         |
| 添加 | addEmp?xxx                | emp---POST        |
| 修改 | updateEmp?id=xxx&xxx=xx   | emp/{id}---PUT    |
| 删除 | deleteEmp?id=1            | emp/{id}---DELETE |

5）、thymeleaf 抽取公共页面：

```html
1、抽取公共片段
    <div th:fragment="copy">
        &copy; 2011 The Good Thymes Virtual Grocery
    </div>

2、引入公共片段
    <div th:insert="~{footer :: copy}"></div>
	~{templatename::selector}：模板名::选择器
	~{templatename::fragmentname}:模板名::片段名
	<!--引入抽取的头部公共部分-->
	<!--模板名：会使用 thymeleaf 的前后缀配置队则进行解析-->
	<div th:insert="~{dashboard :: nav}"></div>

3、默认效果：
	insert的公共片段在div标签中
	如果使用th:insert等属性进行引入，可以不用写~{}：
	行内写法可以加上：[[~{}]];[(~{})]；
```

三种引入公共片段的th属性：

**th:insert**：将公共片段整个插入到声明引入的元素中

**th:replace**：将声明引入的元素替换为公共片段

**th:include**：将被引入的片段的内容包含进这个标签中

```html
<footer th:fragment="copy">
&copy; 2011 The Good Thymes Virtual Grocery
</footer>

引入方式
<div th:insert="footer :: copy"></div>
<div th:replace="footer :: copy"></div>
<div th:include="footer :: copy"></div>

效果
<div>
    <footer>
    &copy; 2011 The Good Thymes Virtual Grocery
    </footer>
</div>

<footer>
	&copy; 2011 The Good Thymes Virtual Grocery
</footer>

<div>
	&copy; 2011 The Good Thymes Virtual Grocery
</div>
```

**引入片段的时候传入参数：** 

```html
<nav class="col-md-2 d-none d-md-block bg-light sidebar" id="sidebar">
    <div class="sidebar-sticky">
        <ul class="nav flex-column">
            <li class="nav-item">
                <a class="nav-link active"
                   th:class="${activeUri=='main.html'?'nav-link active':'nav-link'}"
                   href="#" th:href="@{/main.html}">
                    <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-home">
                        <path d="M3 9l9-7 9 7v11a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2z"></path>
                        <polyline points="9 22 9 12 15 12 15 22"></polyline>
                    </svg>
                    Dashboard <span class="sr-only">(current)</span>
                </a>
            </li>

<!--引入侧边栏;传入参数-->
<div th:replace="commons/bar::#sidebar(activeUri='emps')"></div>
```

提交的数据格式不对问题：生日：日期；

2017-12-12；2017/12/12；2017.12.12；

日期的格式化；SpringMVC将页面提交的值需要转换为指定的类型;

2017-12-12---Date； 类型转换，格式化;

默认日期是按照/的方式；

#### 7、错误处理机制：

**1）、SpringBoot 默认的错误处理机制：** 

默认效果： PC端访问出错，返回一个默认的错误页面。其它客户端则返回一个 json 数据。

原理： 参照 `ErrorMvcAutoConfiguration` 错误处理的自动配置。

​	给容器中添加了以下组件：

​	**a、DefaultErrorAttributes** ：在页面共享信息

```java
@Override
public Map<String, Object> getErrorAttributes(WebRequest webRequest,
        boolean includeStackTrace) {
    Map<String, Object> errorAttributes = new LinkedHashMap<>();
    errorAttributes.put("timestamp", new Date());
    addStatus(errorAttributes, webRequest);
    addErrorDetails(errorAttributes, webRequest, includeStackTrace);
    addPath(errorAttributes, webRequest);
    return errorAttributes;
}
```

​	**b、BasicErrorController** 

```java
@Controller
@RequestMapping("${server.error.path:${error.path:/error}}")
public class BasicErrorController extends AbstractErrorController {
    
    @RequestMapping(produces = MediaType.TEXT_HTML_VALUE)//产生 html 类型的数据，浏览器发送的请求
	public ModelAndView errorHtml(HttpServletRequest request,
			HttpServletResponse response) {
		HttpStatus status = getStatus(request);
		Map<String, Object> model = Collections.unmodifiableMap(getErrorAttributes(
				request, isIncludeStackTrace(request, MediaType.TEXT_HTML)));
		response.setStatus(status.value());
        //去哪个页面作为错误页面；包含页面地址和页面内容
		ModelAndView modelAndView = resolveErrorView(request, response, status, model);
		return (modelAndView != null) ? modelAndView : new ModelAndView("error", model);
	}

	@RequestMapping//产生 json 数据 其它客户端发送的请求
	public ResponseEntity<Map<String, Object>> error(HttpServletRequest request) {
		Map<String, Object> body = getErrorAttributes(request,
				isIncludeStackTrace(request, MediaType.ALL));
		HttpStatus status = getStatus(request);
		return new ResponseEntity<>(body, status);
	}
}
```

​	**c、ErrorPageCustomizer** 

```java
@Value("${error.path:/error}")
private String path = "/error";
//系统出现错误以后来到 error 请求进行处理（类似于 web.xml 注册的错误页面规则）
```

​	**d、DefaultErrorViewResolver** 

```java
@Override
public ModelAndView resolveErrorView(HttpServletRequest request, HttpStatus status,
        Map<String, Object> model) {
    ModelAndView modelAndView = resolve(String.valueOf(status.value()), model);
    if (modelAndView == null && SERIES_VIEWS.containsKey(status.series())) {
        modelAndView = resolve(SERIES_VIEWS.get(status.series()), model);
    }
    return modelAndView;
}

private ModelAndView resolve(String viewName, Map<String, Object> model) {
    //默认SpringBoot可以找到一个页面， error/404
    String errorViewName = "error/" + viewName;
    
    //模板引擎可以解析这个页面地址就用模板引擎解析
    TemplateAvailabilityProvider provider = this.templateAvailabilityProviders
            .getProvider(errorViewName, this.applicationContext);
    if (provider != null) {
        //模板引擎可用的情况下返回到 errorViewName 指定的视图地址
        return new ModelAndView(errorViewName, model);
    }
    //模板引擎不可用，就在静态资源文件夹下找 errorViewName 对应的页面  error/404.html
    return resolveResource(errorViewName, model);
}
```



步骤：

​	一旦系统出现 `4xx` 或者 `5xx` 之类的错误 `ErrorPageCustomizer` 就会生效（定制错误的响应规则）；然后来到 `/error` 请求；该错误请求就会被 `BasicErrorController` 处理；

第一种：响应页面，去哪个页面是由 `DefaultErrorViewResolver` 解析得到的

```java
protected ModelAndView resolveErrorView(HttpServletRequest request,
			HttpServletResponse response, HttpStatus status, Map<String, Object> model) {
    //从所有的 ErrorViewResolver 中得到 ModelAndView
    for (ErrorViewResolver resolver : this.errorViewResolvers) {
        ModelAndView modelAndView = resolver.resolveErrorView(request, status, model);
        if (modelAndView != null) {
            return modelAndView;
        }
    }
    return null;
}
```



**2）、定制错误响应：** 

- 定制错误的页面：

    - 有模板引擎的情况下，默认的错误处理机制回去模板引擎的文件夹下去找 /error/状态码.html 文件。可以在模板 templates 文件夹中创建一个 error 文件夹，将对应的错误页面以状态码命名即可。

        - 可以使用 `4xx` 和 `5xx` 作为错误页面的文件名来匹配这种类型的所有错误，优先寻找精确的状态码页面.html

        - 页面能获取的信息

            **timestamp：** 时间戳

            **status：** 状态码

            **error：** 错误提示

            **exception：** 异常对象

            **message：** 异常消息

            **errors：** JSR303数据校验的错误

    - 没有模板引擎的情况下（模板引找不到对应的文件），默认在静态资源文件夹下寻找对应的状态码.html文件

    - 模板引擎和静态资源文件夹下都没有找到指定的错误页面的时候，默认来到 SpringBoot 的错误提示页面

- 定制错误的 json 数据：

    - 自定义 json 数据并返回

    ```java
    @ControllerAdvice
    public class MyExceptionHandler {
    
        //浏览器和其它客户端返回的都是 json 数据
        @ResponseBody
        @ExceptionHandler(UserNotExistException.class)
        public Map<String,Object> handleException(Exception e){
            Map<String,Object> map = new HashMap<>();
            map.put("code","user.notexist");
            map.put("message",e.getMessage());
            return map;
        }
    }
    //没有自适应效果...
    ```
    - 转发到 /error 进行自适应处理，能出现自适应效果，但不能携带自定义数据

    ```java
    @ExceptionHandler(UserNotExistException.class)
    public String handleException(Exception e, HttpServletRequest request){
        Map<String,Object> map = new HashMap<>();
        //传入我们自己的错误状态码  4xx 5xx，否则就不会进入定制错误页面的解析流程
        /**
         * Integer statusCode=(Integer)request.getAttribute("javax.servlet.error.status_code");
         */
        request.setAttribute("javax.servlet.error.status_code",500);
        map.put("code","user.notexist");
        map.put("message",e.getMessage());
        //转发到/error
        return "forward:/error";
    }
    ```

    - 将我们自定义的数据携带出去：

        当应用程序出现错误以后，会来到 /error 请求，该请求会被 BasicErrorController 处理，响应出去可以获取的数据是由 `getErrorAttributes()` 得到的（是 AbstractErrorController 规定的方法）；

        - 方法一： 完全编写一个 errorController 的实现类，放在容器中；或者编写 AbstractErrorController 的子类，只需要重写其中的方法即可。
        - 方法二： 页面或者 json 返回能用的数据都是通过 `errorAttributes.getErrorAttributes()` 方法得到的。SpringBoot 默认使用 `DefaultErrorAttributes` 来进行数据处理

        ```java
        //给容器中加入我们自己定义的 ErrorAttributes
        @Component
        public class MyErrorAttributes extends DefaultErrorAttributes {
        
            //返回的 map 就是页面和 json 能获取的所有字段
            @Override
            public Map<String, Object> getErrorAttributes(RequestAttributes requestAttributes, boolean includeStackTrace) {
                Map<String, Object> map = super.getErrorAttributes(requestAttributes, includeStackTrace);
                map.put("company","spring boot");
                return map;
            }
        }
        ```

        最终的效果：响应是自适应的，可以通过定制ErrorAttributes改变需要返回的内容。

#### 8、配置嵌入式 Servlet 容器：

SpringBoot 默认使用 Tomcat 作为嵌入式的 Servlet 容器。

##### 1）、定制和修改 Servlet 容器的相关配置：

在 SpringBoot 配置文件中修改和 `server` 有关的配置，其对应的配置类为 `ServerProperties` ：

```properties
server.port=8081
server.servlet.context-path=/

#通用的 servlet 容器设置
server.xxx
#Tomcat 的设置
server.tomcat.uri-encoding=UTF-8
```

SpringBoot 2.0 之前，编写一个**EmbeddedServletContainerCustomizer**：嵌入式的 Servlet 容器的定制器；来修改Servlet容器的配置

```java
@Bean  //一定要将这个定制器加入到容器中
public EmbeddedServletContainerCustomizer embeddedServletContainerCustomizer(){
    return new EmbeddedServletContainerCustomizer() {

        //定制嵌入式的Servlet容器相关的规则
        @Override
        public void customize(ConfigurableEmbeddedServletContainer container) {
            container.setPort(8083);
        }
    };
}

//SpringBoot 2.0之后配置
@Bean
public WebServerFactoryCustomizer webServerFactoryCustomizer(){
    return (WebServerFactoryCustomizer<ConfigurableServletWebServerFactory>) factory -> factory.setPort(8888);
}
```

##### 2）、注册 Servlet 三大组件（Servlet、Filter、Listener）

由于 SpringBoot 默认是以 jar 包的方式启动嵌入式的 Servlet 容器，从而来启动 SpringBoot 的 web 应用。但是在SpringBoot 中并没有 web.xml 文件。所以要注册三大组件需要使用以下方法：

- **注册 Servlet ：** 

    ```java
    //编写 Servlet 实现相应的逻辑
    public class MyServlet extends HttpServlet {
    
        //处理 doGet 请求
        @Override
        protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
            doPost(req, resp);
        }
    
        @Override
        protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
            response.getWriter().write("hello my_servlet");
        }
    }
    ```

    ```java
    @Configuration
    public class MyServerConfig {
    
        //注册 Servlet 三大组件
        @Bean
        public ServletRegistrationBean servletRegistrationBean(){
            //配置编写的 Servlet 并制定访问路径
            return new ServletRegistrationBean<MyServlet>(new MyServlet(), "/my_servlet");
        }
    }
    ```

- **注册 Filter：** 

    ```java
    public class MyFilter implements Filter {
        @Override
        public void init(FilterConfig filterConfig) throws ServletException {
    
        }
    
        @Override
        public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
            System.out.println("过滤器已执行");
            filterChain.doFilter(servletRequest, servletResponse);
        }
    
        @Override
        public void destroy() {
    
        }
    }
    ```

    ```java
    @Bean
    public FilterRegistrationBean<MyFilter> filterRegistrationBean(){
        FilterRegistrationBean<MyFilter> filter = new FilterRegistrationBean<>();
        filter.setFilter(new MyFilter());
        filter.setUrlPatterns(Collections.singletonList("/my_servlet"));
        return filter;
    }
    ```

- **注册 Listener ：** 

    ```java
    public class MyListener implements ServletContextListener {
        @Override
        public void contextInitialized(ServletContextEvent sce) {
            System.out.println("contextInitialized..");
        }
    
        @Override
        public void contextDestroyed(ServletContextEvent sce) {
            System.out.println("contextDestroyed..");
        }
    }
    ```

    ```java
    @Bean
    public ServletListenerRegistrationBean<MyListener> listenerRegistrationBean(){
        return new ServletListenerRegistrationBean<>(new MyListener());
    }
    ```

SpringBoot 在自动配置 SpringMVC 的时候，自动地注册了 SpringMVC 的前端控制器（DispatcherServlet）

```java
@Bean(name = DEFAULT_DISPATCHER_SERVLET_REGISTRATION_BEAN_NAME)
@ConditionalOnBean(value = DispatcherServlet.class, name = DEFAULT_DISPATCHER_SERVLET_BEAN_NAME)
public DispatcherServletRegistrationBean dispatcherServletRegistration(
        DispatcherServlet dispatcherServlet) {
    DispatcherServletRegistrationBean registration = new DispatcherServletRegistrationBean(
            dispatcherServlet, this.webMvcProperties.getServlet().getPath());
    //默认拦截 "/" 所有请求；包括静态资源，但是不拦截 JSP 请求； /* 会拦截 JSP
    //可以通过 server.servletPath 来修改 SpringMVC 前端控制器默认拦截的请求路径
    
    registration.setName(DEFAULT_DISPATCHER_SERVLET_BEAN_NAME);
    registration.setLoadOnStartup(
            this.webMvcProperties.getServlet().getLoadOnStartup());
    if (this.multipartConfig != null) {
        registration.setMultipartConfig(this.multipartConfig);
    }
    return registration;
}
```

##### 3）、SpringBoot 支持其它 Servlet 容器：

SpringBoot 默认支持 Tomcat（默认使用）、Jetty（支持长连接）、Undertow（并发性能高，不支持JSP）

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    引入web模块默认就是使用嵌入式的Tomcat作为Servlet容器；
</dependency>
```

```xml
<!-- 引入web模块 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion>
            <artifactId>spring-boot-starter-tomcat</artifactId>
            <groupId>org.springframework.boot</groupId>
        </exclusion>
    </exclusions>
</dependency>

<!--引入其他的Servlet容器-->
<dependency>
    <artifactId>spring-boot-starter-jetty</artifactId>
    <groupId>org.springframework.boot</groupId>
</dependency>
```

```xml
<!-- 引入web模块 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion>
            <artifactId>spring-boot-starter-tomcat</artifactId>
            <groupId>org.springframework.boot</groupId>
        </exclusion>
    </exclusions>
</dependency>

<!--引入其他的Servlet容器-->
<dependency>
    <artifactId>spring-boot-starter-undertow</artifactId>
    <groupId>org.springframework.boot</groupId>
</dependency>
```

##### 4）、嵌入式 Servlet 容器自动配置原理：

ServletWebServerFactoryConfiguration：嵌入式的 Servlet 容器自动配置

```java
@Configuration
class ServletWebServerFactoryConfiguration {

@Configuration
@ConditionalOnClass({ Servlet.class, Tomcat.class, UpgradeProtocol.class })
@ConditionalOnMissingBean(value = ServletWebServerFactory.class, search = SearchStrategy.CURRENT)
public static class EmbeddedTomcat {

    @Bean
    public TomcatServletWebServerFactory tomcatServletWebServerFactory() {
        return new TomcatServletWebServerFactory();
    }

}
```

以 TomcatServletWebServerFactory 为例：

```java
@Override
public WebServer getWebServer(ServletContextInitializer... initializers) {
    //创建一个Tomcat
    Tomcat tomcat = new Tomcat();
    //配置 Tomcat 的基本环境
    File baseDir = (this.baseDirectory != null) ? this.baseDirectory
            : createTempDir("tomcat");
    tomcat.setBaseDir(baseDir.getAbsolutePath());
    Connector connector = new Connector(this.protocol);
    tomcat.getService().addConnector(connector);
    customizeConnector(connector);
    tomcat.setConnector(connector);
    tomcat.getHost().setAutoDeploy(false);
    configureEngine(tomcat.getEngine());
    for (Connector additionalConnector : this.additionalTomcatConnectors) {
        tomcat.getService().addConnector(additionalConnector);
    }
    prepareContext(tomcat.getHost(), initializers);
    //将配置好的Tomcat传进去，返回一个 WebServer ，并且启动 Tomcat 容器
    return getTomcatWebServer(tomcat);
}
```

**步骤：** 

1、SpringBoot 根据导入的依赖情况给容器中添加相应的嵌入式容器工厂：

```java
@Configuration
class ServletWebServerFactoryConfiguration {
	@Configuration
	@ConditionalOnClass({ Servlet.class, Tomcat.class, UpgradeProtocol.class })
	@ConditionalOnMissingBean(value = ServletWebServerFactory.class, search = SearchStrategy.CURRENT)
	public static class EmbeddedTomcat {

		@Bean
		public TomcatServletWebServerFactory tomcatServletWebServerFactory() {
			return new TomcatServletWebServerFactory();
		}

	}
}
```

2、当容器总创建某个组件后，后置处理器就会工作： **BeanPostProcessorsRegistrar** 

```java
public static class BeanPostProcessorsRegistrar
			implements ImportBeanDefinitionRegistrar, BeanFactoryAware {
    @Override
		public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata,
				BeanDefinitionRegistry registry) {
			if (this.beanFactory == null) {
				return;
			}
			registerSyntheticBeanIfMissing(registry,
					"webServerFactoryCustomizerBeanPostProcessor",
					WebServerFactoryCustomizerBeanPostProcessor.class);
			registerSyntheticBeanIfMissing(registry,
					"errorPageRegistrarBeanPostProcessor",
					ErrorPageRegistrarBeanPostProcessor.class);
		}
}
```

只要是嵌入式的 Servlet 容器工厂，后置处理器就会工作。

3、后置处理器会从容器中获取所有的定制器，并调用定制器的定制方法：

```java
private Collection<WebServerFactoryCustomizer<?>> getCustomizers() {
    if (this.customizers == null) {
        // Look up does not include the parent context
        this.customizers = new ArrayList<>(getWebServerFactoryCustomizerBeans());
        this.customizers.sort(AnnotationAwareOrderComparator.INSTANCE);
        this.customizers = Collections.unmodifiableList(this.customizers);
    }
    return this.customizers;
}
```

##### 5）、嵌入式 Servlet 容器启动原理：

什么时候创建嵌入式 Servlet 容器工厂，什么时候获取嵌入式 Servlet 容器并启动 Tomcat ；

获取嵌入式的 Servlet 容器工厂：

1、SpringBoot 应用启动运行 run 方法

2、SpringBoot 刷新 IOC 容器（创建 IOC 容器对象并初始化容器，创建容器中的每一个组件）

3、刷新刚才创建好的 IOC 容器

4、onRefresh()：web 的 IOC 容器重写了 onRefresh 方法

5、web IOC 容器会创建嵌入式的 Servlet 容器

6、获取嵌入式的 Servlet 容器工厂 `ServletWebServerFactory factory = getWebServerFactory();` 从 IOC 容器中获取 tomcatServletWebServerFactory 创建对象，后置处理器工作，获取所有的定制器来定制 Servlet 容器的相关配置

7、使用容器工厂获取嵌入式的 Servlet 容器：this.webServer = factory.getWebServer(getSelfInitializer());

8、嵌入式的 Servlet 容器创建对象，并启动 Servlet 容器；

请启动嵌入式的 Servlet 容器，然后再将 ioc 容器中剩下没有创建的对象获取出来。 IOC 容器启动的时候就会创建嵌入式 Servlet 容器。

#### 9、使用外部 Servlet 容器：

嵌入式 Servlet 容器： 应用打包成可执行的 jar 包

​	优点：简单、便携

​	缺点：默认不支持 JSP、优化和定制比较复杂（使用定制器【ServerProperties、自定义WebServerFactoryCustomizer】）

外置的 Servlet 容器： 使用外部 Tomcat ---> 应用以 war 方式打包

**步骤：** 

1）、必须创建一个 war 项目；

2）、将嵌入式的 Tomcat 指定为 `provided` 

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-tomcat</artifactId>
    <scope>provided</scope>
</dependency>
```

3）、必须编写一个 `SpringBootServletInitializer` 的子类，并调用 configure 方法

```java
public class ServletInitializer extends SpringBootServletInitializer {

    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {
        //传入 SpringBoot 的应用主程序
        return application.sources(Springboot20190114WebApplication.class);
    }

}
```

4）、启动外部服务器就可以使用了

**原理：** 

jar 包方式： 执行 SpringBoot 主类的 main 方法，启动 IOC 容器，创建嵌入式的 Servlet 容器；

war 包方式：启动服务器， **服务器启动 SpringBoot 应用（SpringBootServletInitializer）** ，启动 IOC 容器；



Servlet3.0

8.2.4 Shared libraries / runtimes pluggability：

规则：

​	1）、服务器启动（web应用启动）会创建当前web应用里面每一个jar包里面ServletContainerInitializer实例：

​	2）、ServletContainerInitializer的实现放在jar包的META-INF/services文件夹下，有一个名为javax.servlet.ServletContainerInitializer的文件，内容就是ServletContainerInitializer的实现类的全类名

​	3）、还可以使用@HandlesTypes，在应用启动的时候加载我们感兴趣的类；

**流程：** 

1）、启动 Tomcat 服务器；

2）、org\springframework\spring-web\5.1.4.RELEASE\spring-web-5.1.4.RELEASE.jar!\META-INF\services\javax.servlet.ServletContainerInitializer；

在 Spring 的 web 模块中有该文件，其内容为： org.springframework.web.SpringServletContainerInitializer

3）、SpringServletContainerInitializer将 @HandlesTypes(WebApplicationInitializer.class) 标注的所有这个类型的类都传入到onStartup方法的Set<Class<?>>；为这些 WebApplicationInitializer 类型的类创建实例；

4）、每一个 WebApplicationInitializer 都会调用自己的 onStartup 方法

5）、相当于 SpringBootServletInitializer 的类会被创建对象，并执行onStartup方法

6）、SpringBootServletInitializer 实例执行 onStartup 的时候会 createRootApplicationContext；创建容器

```java
protected WebApplicationContext createRootApplicationContext(
			ServletContext servletContext) {
     //1、创建SpringApplicationBuilder
    SpringApplicationBuilder builder = createSpringApplicationBuilder();
    builder.main(getClass());
    ApplicationContext parent = getExistingRootWebApplicationContext(servletContext);
    if (parent != null) {
        this.logger.info("Root context already created (using as parent).");
        servletContext.setAttribute(
                WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE, null);
        builder.initializers(new ParentContextApplicationContextInitializer(parent));
    }
    builder.initializers(
            new ServletContextApplicationContextInitializer(servletContext));
    builder.contextClass(AnnotationConfigServletWebServerApplicationContext.class);
    //调用configure方法，子类重写了这个方法，将SpringBoot的主程序类传入了进来
    builder = configure(builder);
    builder.listeners(new WebEnvironmentPropertySourceInitializer(servletContext));
    
    //使用builder创建一个Spring应用
    SpringApplication application = builder.build();
    if (application.getAllSources().isEmpty() && AnnotationUtils
            .findAnnotation(getClass(), Configuration.class) != null) {
        application.addPrimarySources(Collections.singleton(getClass()));
    }
    Assert.state(!application.getAllSources().isEmpty(),
            "No SpringApplication sources have been defined. Either override the "
                    + "configure method or add an @Configuration annotation");
    // Ensure error pages are registered
    if (this.registerErrorPageFilter) {
        application.addPrimarySources(
                Collections.singleton(ErrorPageFilterConfiguration.class));
    }
    //启动Spring应用
    return run(application);
}
```

7）、Spring的应用启动并且创建 IOC 容器

## 五、Docker

#### 1、简介：

Docker 是一个开源的应用容器引擎；Docker支持将软件编译成一个镜像；然后在镜像中将各种软件做好配置，将镜像发布出去，其他使用者可以直接使用这个镜像；运行中的这个镜像被称为容器，容器启动是非常快的。

#### 2、核心概念：

- **docker 主机（Host）：** 安装了 Docker 程序的机器，一个物理或者虚拟的机器用于执行 Docker 守护进程和容器。
- **docker 客户端（Client）：** 连接 Docker 主机进行操作
- **docker 仓库（Registry）：** 用来保存各种打包好的软件镜像
- **docker 镜像（Images）：** 软件打包好的镜像，放在 docker 仓库中
- **docker 容器（Container）：** 镜像启动后的实例称为一个容器，容器是独立运行的一个或一组应用

使用 Docker 的步骤：

1）、安装 Docker

2）、去 Docker 仓库找到软件镜像

3）、使用 Docker 运行镜像，这个镜像就会生成一个 Docker 容器

4）、对容器的启动停止就是对软件的启动停止

#### 3、安装 Docker：

1）、在虚拟机上安装 Docker：

**步骤：** 

- 检查内核版本，必须是3.10版本及以上

    ```shell
    [root@localhost local]# uname -r
    ```

    如果内核版本过低可以使用 `yum update` 命令升级，谨慎操作！

- 安装docker

    ```shell
    [root@localhost local]# yum install docker
    ```

- 输入 y 确认安装

- 启动 docker

    ```shell
    [root@localhost local]# systemctl start docker
    ```

- 查看 docker 版本号

    ```shell
    [root@localhost local]# docker -v
    Docker version 1.13.1, build 07f3374/1.13.1
    ```

- 设置开机启动 docker 

    ```shell
    [root@localhost local]# systemctl enable docker
    ```

- 停止 docker

    ```shell
    [root@localhost local]# systemctl stop docker
    ```

#### 4、Docker 常用命令和操作：

##### 1）、镜像操作：

| 操作 |                      命令                       |                           说明                           |
| :--- | :---------------------------------------------: | :------------------------------------------------------: |
| 检索 | docker  search 关键字  eg：docker  search redis | 我们经常去docker  hub上检索镜像的详细信息，如镜像的TAG。 |
| 拉取 |             docker pull 镜像名:tag              | :tag是可选的，tag表示标签，多为软件的版本，默认是latest  |
| 列表 |                  docker images                  |                     查看所有本地镜像                     |
| 删除 |               docker rmi image-id               |                    删除指定的本地镜像                    |

以 mysql 为例： [Docker Hub](https://hub.docker.com/)

- 去仓库中检索 mysql 

    ```shell
    [root@localhost local]# docker search mysql
    ```

    检索完成后会显示一个结果列表，其中 `INDEX` 表示索引， `NAME` 为组件名称， `DESCRIPTION` 为描述，`STARS` 表示使用人数， `OFFICIAL` 表示是否为官方镜像， `AUTOMATED` 表示是否自动配置。

- 拉取镜像： 使用 `docker pull 镜像名:tag` 的命令来拉取镜像， :tag 表示的是镜像版本

    ```shell
    [root@localhost local]# docker pull mysql
    ```

- 查看镜像列表：

    ```shell
    [root@localhost local]# docker images
    ```

- 删除指定的镜像：

    ```shell
    [root@localhost local]# docker rmi 102816b1ee7d
    ```

##### 2）、容器操作：

运行镜像就会产生一个容器，产生的容器才是正在运行的软件

|   操作   |                             命令                             |                            说明                             |
| :------: | :----------------------------------------------------------: | :---------------------------------------------------------: |
|   运行   |     docker  run  --name  container-name  -d  image-name      | --name：自定义容器名；-d 后台运行；image-name：指定镜像模板 |
|   列表   |                docker  ps（查看运行中的容器）                |                  加上 -a 可以查看所有容器                   |
|   停止   |          docker  stop  container-name/container-id           |                     停止当前运行的容器                      |
|   启动   |          docker  start  container-name/container-id          |                          启动容器                           |
|   删除   |                   docker  rm  container-id                   |                       删除指定的容器                        |
| 端口映射 | -p  6379:6379        eg:  docker  run  -d  -p  6379:6379  --name  myredis  docker.io/redis |            -p：主机端口（映射到）容器内部的端口             |
| 容器日志 |          docker  logs  container-name/container-id           |                                                             |

- 运行 docker 镜像：

    ```shell
    [root@localhost local]# docker run --name mytomcat -d tomcat:latest
    ```

- 查看运行中的容器

    ```shell
    [root@localhost local]# docker ps
    ```

- 停止运行中的容器

    ```shell
    [root@localhost local]# docker stop mytomcat
    或者
    [root@localhost local]# docker stop b0e06c062a90
    ```

- 查看所有的容器包括运行中的和退出的

    ```shell
    [root@localhost local]# docker ps -a
    ```

- 启动已经停止的容器

    ```shell
    [root@localhost local]# docker start b0e06c062a90
    ```

- 删除指定的容器： 该容器的状态为停止状态

    ```shell
    [root@localhost local]# docker rm b0e06c062a90
    ```

- 启动关联端口映射的容器              主机端口:容器内部端口

    ```shell
    [root@localhost local]# docker run -d -p 8081:8080 --name mytomcat tomcat
    ```

- 查看容器日志

    ```shell
    [root@localhost local]# docker logs mytomcat
    ```

## 六、SpringBoot 与数据访问

##### 1、JDBC：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>

<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <scope>runtime</scope>
</dependency>
```

数据库配置：

```yaml
spring:
  datasource:
    url: jdbc:mysql://192.168.174.131:3306/jdbc
    username: root
    password: root
    driver-class-name: com.mysql.cj.jdbc.Driver
```

SpringBoot 2.0 之后默认使用 HikariDataSource 作为数据源；数据源的相关配置都在 `DataSourceProperties` 中

自动配置原理：    org.springframework.boot.autoconfigure.jdbc

- 参考 DataSourceConfiguration ，根据配置创建数据源。默认使用 `HikariDataSource` 连接池，可以使用 spring.datasource.type 指定自定义的数据源类型；

- SpringBoot 默认支持的数据源：

    ```java
    org.apache.tomcat.jdbc.pool.DataSource
    org.apache.commons.dbcp2.BasicDataSource
    com.zaxxer.hikari.HikariDataSource
    ```

    ```java
    //自定义数据源
    @ConditionalOnMissingBean(DataSource.class)
    @ConditionalOnProperty(name = "spring.datasource.type")
    static class Generic {
    
        @Bean
        public DataSource dataSource(DataSourceProperties properties) {
            //使用DataSourceBuilder创建数据源，利用反射创建相应type的数据源并且绑定相关属性
            return properties.initializeDataSourceBuilder().build();
        }
    
    }
    ```


##### 2、整合 Druid 数据库连接池：

- 导入 druid 依赖：

    ```xml
    
    ```

- 整合 druid ：

    ```java
    @Configuration
    public class DruidConfig {
    
        @ConfigurationProperties(prefix = "spring.datasource")
        @Bean
        public DataSource druid(){
           return  new DruidDataSource();
        }
    
        //配置Druid的监控
        //1、配置一个管理后台的Servlet
        @Bean
        public ServletRegistrationBean statViewServlet(){
            ServletRegistrationBean bean = new ServletRegistrationBean(new StatViewServlet(), "/druid/*");
            Map<String,String> initParams = new HashMap<>();
    
            initParams.put("loginUsername","admin");
            initParams.put("loginPassword","123456");
            initParams.put("allow","");//默认就是允许所有访问
            initParams.put("deny","192.168.15.21");
    
            bean.setInitParameters(initParams);
            return bean;
        }
    
        //2、配置一个web监控的filter
        @Bean
        public FilterRegistrationBean webStatFilter(){
            FilterRegistrationBean bean = new FilterRegistrationBean();
            bean.setFilter(new WebStatFilter());
    
            Map<String,String> initParams = new HashMap<>();
            initParams.put("exclusions","*.js,*.css,/druid/*");
    
            bean.setInitParameters(initParams);
    
            bean.setUrlPatterns(Arrays.asList("/*"));
    
            return  bean;
        }
    }
    ```

##### 3、整合 Mybatis：

- 添加数据库驱动、jdbc、druid连接池、Mybatis 依赖：

    ```xml
    <!--驱动-->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <scope>runtime</scope>
    </dependency>
    <!--jdbc-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-jdbc</artifactId>
    </dependency>
    <!--druid-->
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>druid-spring-boot-starter</artifactId>
        <version>1.1.10</version>
    </dependency>
    <!--mybatis-->
    <dependency>
        <groupId>org.mybatis.spring.boot</groupId>
        <artifactId>mybatis-spring-boot-starter</artifactId>
        <version>1.3.2</version>
    </dependency>
    ```

- 创建数据库、javaBean

- **Mybatis 注解版：** 

    ```java
    //指定这是一个操作数据库的mapper
    @Mapper
    public interface DepartmentMapper {
    
        @Select("select * from department where id = #{id}")
        Department getDeptById(Integer id);
    
        @Delete("delete from department where id = #{id} ")
        int deleteDeptById(Integer id);
    
        @Options(useGeneratedKeys = true, keyProperty = "id")//获取自增主键
        @Insert("insert into department(departmentName) values (#{departmentName}) ")
        int insertDept(Department department);
    
        @Update("update department set departmentName = #{departmentName} where id = #{Id} ")
        int updateDept(Department department);
    }
    ```

- **自定义 Mybatis 的配置规则：** 

    ```java
    @org.springframework.context.annotation.Configuration
    public class MybatisConfig {
    
        @Bean
        public ConfigurationCustomizer configurationCustomizer(){
            return new ConfigurationCustomizer(){
    
                @Override
                public void customize(Configuration configuration) {
                	//开启驼峰命名规则
                    configuration.setMapUnderscoreToCamelCase(true);
                }
            };
        }
    }
    ```

    在主配置类上添加 `@MapperScan(value = "com.yunhui.mapper")` 注解指定扫描 Mapper 的路径，这样在 Mapper 的接口文件上就不用添加 `@Mapper` 注解。

- **Mybatis 配置文件版：** 

    ```yaml
    # yml文件中配置 mybatis配置文件路径，sql映射文件路径
    mybatis:
      config-location: classpath:mybatis/mybatis-config.xml
      mapper-locations: classpath:mybatis/mapper/*.xml
    ```

##### 4、SpringData JPA：

JPA： ORM，步骤：

- 编写一个实体类（bean）和数据表进行映射，并且配置好映射关系

    ```java
    //使用JPA注解配置映射关系
    @Entity             //告诉JPA这是一个实体类（和数据表映射的类）
    @Table(name = "user")   //@Table 指定和哪个数据表对应，如果省略默认表名就是类名的小写
    public class User {
    
        @Id//标注该字段为主键
        @GeneratedValue(strategy = GenerationType.IDENTITY)//设置主键自增
        private Integer id;
    
        @Column(name = "last_name", length = 25)//这是和数据表对应的一个列
        private String lastName;
    
        @Column(name = "email", length = 50)//如果省略默认列名就是属性名
        private String email;
    }
    ```

- 编写个 Dao 接口来操作实体类对应的数据表（SpringData中称为 Repository）

    ```java
    //继承 JpaRepository 来完成数据库的操作
    //泛型： <要操作的实体类，主键类型>
    public interface UserRepository extends JpaRepository<User, Integer> {
        
    }
    ```

- 基本配置

    ```yaml
    spring:
      jpa:
        hibernate:
    #     update：更新或者创建数据表
          ddl-auto: update
    #   控制台显示 SQL
        show-sql: true
    ```

- SpringBoot JPA 查询新写法：

    ```java
    @GetMapping("/user/{id}")
    public User getUser(@PathVariable("id") Integer id){
        User user = new User();
        user.setId(id);
        Example<User> example = Example.of(user);
        return userRepository.findOne(example).orElse(null);
    }
    ```

## 七、SpringBoot 启动配置原理

几个重要的事件回调机制

配置在META-INF/spring.factories文件中

**ApplicationContextInitializer** 

**SpringApplicationRunListener** 



只需要放在ioc容器中

**ApplicationRunner** 

**CommandLineRunner** 

启动流程：

##### 1、创建 SpringApplication 对象：

```java
//创建对象
public SpringApplication(Class<?>... primarySources) {
    this(null, primarySources);
}

@SuppressWarnings({ "unchecked", "rawtypes" })
public SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources) {
    this.resourceLoader = resourceLoader;
    Assert.notNull(primarySources, "PrimarySources must not be null");
    //保存主配置类
    this.primarySources = new LinkedHashSet<>(Arrays.asList(primarySources));
    //判断当前是否是一个 web 应用
    this.webApplicationType = WebApplicationType.deduceFromClasspath();
    //从类路径下找到 META-INF/spring.factories 里配置的所有 ApplicationContextInitializer 然后保存起来
    setInitializers((Collection) getSpringFactoriesInstances(
            ApplicationContextInitializer.class));
    //从类路径下找到 META-INF/spring.factories 里配置的所有 ApplicationListener
    setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));
    //从多个配置类中找到有 main 方法的主配置类
    this.mainApplicationClass = deduceMainApplicationClass();
}
```

##### 2、运行 run 方法：

```java
public ConfigurableApplicationContext run(String... args) {
    StopWatch stopWatch = new StopWatch();
    stopWatch.start();
    //声明一个 IOC 容器
    ConfigurableApplicationContext context = null;
    Collection<SpringBootExceptionReporter> exceptionReporters = new ArrayList<>();
    configureHeadlessProperty();
    //获取 SpringApplicationRunListeners 从类路径下 META-INF/spring.factories 
    SpringApplicationRunListeners listeners = getRunListeners(args);
    //回调获取的所有 SpringApplicationRunListener 的 starting() 方法
    listeners.starting();
    try {
        //封装命令行参数
        ApplicationArguments applicationArguments = new DefaultApplicationArguments(
                args);
        //准备环境
        ConfigurableEnvironment environment = prepareEnvironment(listeners,
                applicationArguments);
        	//创建环境完成后会回调所有的 SpringApplicationRunListeners 的environmentPrepared方法
        	//表示环境准备完成
        
        configureIgnoreBeanInfo(environment);
        //打印spring-boot图标
        Banner printedBanner = printBanner(environment);
        //创建一个 IOC 容器；决定创建 web 的 IOC 还是普通的 IOC
        context = createApplicationContext();
        exceptionReporters = getSpringFactoriesInstances(
                SpringBootExceptionReporter.class,
                new Class[] { ConfigurableApplicationContext.class }, context);
        //准备上下文环境
        prepareContext(context, environment, listeners, applicationArguments,
                printedBanner);
        	//将 environment 保存到 IOC 中
        	//applyInitializers(context);回调之前保存的所有 ApplicationContextInitializer的
        	//initialize 方法，还要回调所有的 SpringApplicationRunListener 的contextPrepared()方法
        	//prepareContext 完全运行完成以后回调所有SpringApplicationRunListener的contextLoaded方法
        
        //刷新容器； IOC 容器初始化，加载所有组件，如果是 web 应用还会创建嵌入式的 Tomcat
        refreshContext(context);
        
        //从 IOC 容器中获取所有的 ApplicationRunner 和 CommandLineRunner
        //先回调ApplicationRunner，再回调CommandLineRunner
        afterRefresh(context, applicationArguments);
        
        stopWatch.stop();
        if (this.logStartupInfo) {
            new StartupInfoLogger(this.mainApplicationClass)
                    .logStarted(getApplicationLog(), stopWatch);
        }
        listeners.started(context);
        callRunners(context, applicationArguments);
    }
    catch (Throwable ex) {
        handleRunFailure(context, ex, exceptionReporters, listeners);
        throw new IllegalStateException(ex);
    }

    try {
        listeners.running(context);
    }
    catch (Throwable ex) {
        handleRunFailure(context, ex, exceptionReporters, null);
        throw new IllegalStateException(ex);
    }
    //整个SpringBoot启动完成后返回启动的 IOC 容器
    return context;
}
```

##### 3、事件监听机制：

配置在 META-INF/spring.factories

**ApplicationContextInitializer** 

```java
public class HelloApplicationContextInitializer implements ApplicationContextInitializer<ConfigurableApplicationContext> {
    @Override
    public void initialize(ConfigurableApplicationContext applicationContext) {
        System.out.println("ApplicationContextInitializer...initialize" + applicationContext);
    }
}
```

**SpringApplicationRunListener** 

```java
public class HelloSpringApplicationRunListener implements SpringApplicationRunListener {

    //必须要有有参构造器
    public HelloSpringApplicationRunListener(SpringApplication application, String[] args) {

    }

    @Override
    public void starting() {
        System.out.println("SpringApplicationRunListener...starting");
    }

    @Override
    public void environmentPrepared(ConfigurableEnvironment environment) {
        Object o = environment.getSystemProperties().get("os.name");
        System.out.println(o);
        System.out.println("SpringApplicationRunListener...environmentPrepared");
    }

    @Override
    public void contextPrepared(ConfigurableApplicationContext context) {
        System.out.println("SpringApplicationRunListener...contextPrepared");
    }

    @Override
    public void contextLoaded(ConfigurableApplicationContext context) {
        System.out.println("SpringApplicationRunListener...contextLoaded");
    }

    @Override
    public void started(ConfigurableApplicationContext context) {
        System.out.println("SpringApplicationRunListener...started");
    }

    @Override
    public void running(ConfigurableApplicationContext context) {
        System.out.println("SpringApplicationRunListener...running");
    }

    @Override
    public void failed(ConfigurableApplicationContext context, Throwable exception) {
        System.out.println("SpringApplicationRunListener...failed");
    }
}
```

配置（META-INF/spring.factories）

```properties
org.springframework.context.ApplicationContextInitializer=\
com.yunhui.listener.HelloApplicationContextInitializer

org.springframework.boot.SpringApplicationRunListener=\
com.yunhui.listener.HelloSpringApplicationRunListener
```

只需要放在ioc容器中

**ApplicationRunner** 

```java
@Component
public class HelloApplicationRunner implements ApplicationRunner {
    @Override
    public void run(ApplicationArguments args) throws Exception {
        System.out.println("ApplicationRunner...run");
    }
}
```

**CommandLineRunner** 

```java
@Component
public class HelloCommandLineRunner implements CommandLineRunner {
    @Override
    public void run(String... args) throws Exception {
        System.out.println("CommandLineRunner...run..." + Arrays.asList(args));
    }
}
```

## 八、自定义start

自定义start需要：

1、这个场景需要导入的依赖是什么？

2、如何编写自动配置

```java
@Configuration	//指定这个类是一个配置类
@ConditionalOnxxx	//在指定条件成立的情况下自动配置类生效
@AutoConfigureAfter	//指定自动配置类的顺序
@Bean	//给容器中添加组件

@ConfigurationPropertie结合相关xxxProperties类来绑定相关的配置
@EnableConfigurationProperties //让xxxProperties生效加入到容器中

自动配置类要能加载
将需要启动就加载的自动配置类，配置在META-INF/spring.factories
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration,\
org.springframework.boot.autoconfigure.aop.AopAutoConfiguration
```

3、模式：

启动器只用来做依赖导入；

专门来写一个自动配置模块；

启动器依赖自动配置；别人只需要引入启动器（starter）

mybatis-spring-boot-starter；自定义启动器名-spring-boot-starter



