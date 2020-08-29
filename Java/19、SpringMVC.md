## SpringMVC

#### 1、MVC思想：

WEB开发中需要根据功能职责的不同划分为控制层、业务层、持久层。

**控制层：** 和用户界面做交互        **业务层：** 处理应用业务逻辑        **持久层：** 连接和操作数据库

**Model（模型）：** 数据模型，包含要展示的数据和业务功能；

**View（视图）：** 用户界面，在界面上显示模型数据；

**Controller（控制器）：** 调度作用，接收用户请求、调用业务处理请求、共享模型数据并跳转页面

#### 2、前端控制器：

要求在 web 开发的最前面设置一个入口的控制器，用来集中处理请求的机制，所有的请求都应该发往前端控制器，进行统一的处理，然后前端控制器再把每个请求分发给各自的处理器。

作用：减少重复代码，权限检查、授权操作、日志记录等。

#### 3、SpringMVC配置：

- **配置前端控制器：** 

  ```xml
  <servlet>
      <servlet-name>dispatcherServlet</servlet-name>
      <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
      <!--
      指定spring-mvc配置文件的路径，如果不指定需要在web.xml同级目录在新建 dispatcherServlet-servlet.xml 文件
      命名要求， servlet名加上"-servlet"
      <init-param>
          <param-name>contextConfigLocation</param-name>
          <param-value>classpath:</param-value>
      </init-param>
          -->
      <load-on-startup>1</load-on-startup>
  </servlet>
  <servlet-mapping>
      <servlet-name>dispatcherServlet</servlet-name>
      <url-pattern>/</url-pattern>
  </servlet-mapping>
  ```

- **在 springMVC 配置文件中配置处理器映射器、处理器适配器、视图解析器：** 

  ```xml	
  <!--
      处理器映射器：BeanNameUrlHandlerMapping
      目的：选择哪一个处理器来处理（Controller）当前请求
      根据请求的 URL 去寻找对应的 Bean
      /hello    去匹配 id 或 name 为 hello 的bean
  -->
  <bean class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping"/>
  
  <!--
      处理器适配器：SimpleControllerHandlerAdapter
      目的：调用处理器（Controller）的处理请求方法
          1、所有的适配器都实现 HandlerAdapter 接口
          2、处理器（Controller）类必须实现 org.springframework.web.servlet.mvc.Controller
  -->
  <bean class="org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter"/>
  
  <!--
      视图解析器：InternalResourceViewResolver
      处理视图
  -->
  <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
      <property name="prefix" value="/WEB-INF/views/"/>
      <property name="suffix" value=".jsp"/>
  </bean>
  
  <!--
      Handler 处理器
      在 SpringMVC 中 Handler 和 Controller 是同一个东西
  -->
  <bean name="/hello" class="com.spring.hello.HelloController"/>
  ```

#### 4、SpringMVC执行流程：

>1）、**发送请求：** 客户端发送 Http 请求，请求被前端控制器（DispatcherServlet）拦截进行处理
>
>2）、**查找处理器：** 前端控制器接收请求之后会通过处理器映射器（HandlerMapping）去查找处理器，寻找当前要调用哪一个处理器
>
>3）、**返回处理器执行链：** 处理器映射器找到Controller后向前端控制器返回处理器的执行链（HandleExecutorChain），包含拦截器和当前找到的 Handle 对象，即Controller
>
>4）、**处理适配：** 前端控制器去寻找处理器适配器（HandlerAdapter）
>
>5）、**执行处理器方法：** 调用处理器对象又称之为 Controller ，执行具体方法
>
>6）、**返回 ModelAndView 对象：** 处理器执行完成后会向处理适配器返回视图对象
>
>7）、**处理适配器返回：** 处理器适配器（HandlerAdapter）将接收的ModelAndView对象返回给前端控制器
>
>8）、**视图解析：** 前端控制器调用视图解析器（ViewResolver）做视图的解析操作
>
>9）、**返回 view 视图：** 视图解析器解析完成后返回一个view视图
>
>10）、**渲染视图：** 前端控制把Model数据填充到view视图中完成视图的渲染
>
>11）、**返回响应：** 完成视图渲染后给客户端返回一个响应

 ![SpringMVC执行流程](G:\云汇科技\笔记\浙思云汇\assets/SpringMVC-1544538265216.png)

#### 5、SpringMVC核心组件：

- **1）、前端控制器（DispatcherServlet）：** 

  不需要程序员开发，由框架提供，需要在web.xml中配置。

  **作用：** 接收请求，处理相应结果，相当于一个中央处理器

- **2）、处理器映射器（HandlerMapping）：** 

  不需要程序员开发，由框架提供。

  **作用：** 根据请求的 URL 找到对应的 Handler 并返回处理器执行链（HandlerExecutorChain）

- **3）、处理器适配器（HandlerAdapter）：** 

  不需要程序员开发，由框架提供。

  **作用：** 调用处理器（Handler/Controller）的方法

- **4）、处理器（Handler）：** 后端控制器

  又叫做 Controller ，需要程序员开发，必须按照 HandlerAdapter 的规范去开发

  **作用：** 接收用户的请求数据，调用业务方法处理请求。

- **5）、视图解析器（ViewResolver）：** 

  不需要程序员开发，由框架或第三方提供。

  **作用：** 视图解析，把逻辑视图名称解析成真正的物理视图，SpringMVC 支持多种视图技术。

- **6）、视图（View）：** 

  把数据展现给用户。

#### 6、使用注解开发：

使用 XML 配置开发的弊端：

配置比较繁琐；Controller 必须实现 Controller 接口，如果要处理多个请求，得开发多个 Controller。

使用注解开发不需要手动配置 **处理器映射器（HandlerMapping）** 、 **处理器适配器（HandlerAdapter）** 、**视图解析器（ViewResolver）** 。SpringMVC 中有默认设置。org/springframework/web/servlet/DispatcherServlet.properties文件中有默认配置信息。

```properties
# Default implementation classes for DispatcherServlet's strategy interfaces.
# Used as fallback when no matching beans are found in the DispatcherServlet context.
# Not meant to be customized by application developers.

org.springframework.web.servlet.LocaleResolver=org.springframework.web.servlet.i18n.AcceptHeaderLocaleResolver

org.springframework.web.servlet.ThemeResolver=org.springframework.web.servlet.theme.FixedThemeResolver

org.springframework.web.servlet.HandlerMapping=org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping,\
	org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping

org.springframework.web.servlet.HandlerAdapter=org.springframework.web.servlet.mvc.HttpRequestHandlerAdapter,\
	org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter,\
	org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter

org.springframework.web.servlet.HandlerExceptionResolver=org.springframework.web.servlet.mvc.method.annotation.ExceptionHandlerExceptionResolver,\
	org.springframework.web.servlet.mvc.annotation.ResponseStatusExceptionResolver,\
	org.springframework.web.servlet.mvc.support.DefaultHandlerExceptionResolver

org.springframework.web.servlet.RequestToViewNameTranslator=org.springframework.web.servlet.view.DefaultRequestToViewNameTranslator

org.springframework.web.servlet.ViewResolver=org.springframework.web.servlet.view.InternalResourceViewResolver

org.springframework.web.servlet.FlashMapManager=org.springframework.web.servlet.support.SessionFlashMapManager
```

使用注解开发需要在 springMVC 的配置文件中开启注解扫描：

```xml
<!-- MVC注解解析器
 	能够自动注册RequestMappingHandlerMapping  RequestMappringHandlerAdapter ExceptionHandlerExceptionResolver三个bean
-->
<mvc:annotation-driven/>
```

#### 7、@RequestMapping注解：

用来处理请求地址映射的注解，可以添加在类上或者方法上。添加在类上表示类中所有的请求方法 URL 都必须把类上的 URL 地址作为父路径。

从 Spring4.3 开始，针对不同的请求新增了几个功能和 @RequestMapping 相同的注解。

- `@GetMapping` 
- `@PostMapping`
- `@PutMapping`
- `@DeleteMapping`
- `@PatchMapping`

**属性：**

- **value：** 和 **path** 一样，用来指定请求的 URL。/hello
- **method：** 指定请求方式的类型，请求方式的类型：GET, HEAD, POST, PUT, PATCH, DELETE, OPTIONS, TRACE
- **consumes：** 指定请求的内容格式：Content-Type=application/json
- **produces：** 指定响应的内容格式：application/json;charset=UTF-8
- **params：** 指定 request 中必须包含某些参数值，包含该方法才处理
- **headers：** 指定 request 中必须包含指定的 header

#### 8、SpringMVC静态资源访问：

**前端控制器的 url-pattern 配置：**

- 第一种： *.拓展名（\*.do），不会导致静态资源被拦截的问题，但不支持 RESTfull 编码风格。
- 第二种： /，支持 RESTfull 风格，但会导致静态资源被拦截。

**静态资源被拦截的问题原因：** 

在 Tomcat 中处理静态资源的 servlet 叫 default ，而 default 映射的路径就是 / 。启动项目的时候，Tomcat 就会加载自己的全局配置文件 web.xml ，然后再加载项目的 web.xml 文件。这样项目中的 web.xml 所配置的 / 映射覆盖了 Tomcat 中 web.xml 中配置的映射规则。所以不能访问静态资源。

**解决方法：** 

- 使用标签： \<mvc:default-servlet-handler/> 在开发中使用比较多。

  原理：将 SpringMVC 上下文中定义一个 DefaultServletHttpRequestHandler 类，会对所有前端控制器的请求做筛选和盘查，如果发现没有经过映射请求，即不是 Controller ，就交给 Tomcat 的默认 Servlet 来处理。

- 资源映射：

  ```xml
  <!--
  	location：指定静态资源路径，是一个 Resouce 类型，可以使用前缀，如classpath:
  	mapping：表示映射路径
  -->
  <mvc:resources mapping="/**" location="/"/>
  ```

#### 9、请求和响应：

- **请求转发：** forward:/hello.html

  浏览器地址栏不改变，并可以共享请求中的数据。

- **URL重定向：** redirect:/hello.html

  浏览器地址栏会改变，不能共享请求中的数据

|              |   请求转发   |   URL重定向    |
| :----------: | :----------: | :------------: |
|    地址栏    |   不会改变   |     会改变     |
|   共享数据   | 可以共享数据 | 一般情况下不能 |
| 表单重复提交 |    会导致    |    不会导致    |

Spring3.1之后提供了一个 Flash 属性，用于重定向之间传递数据，只能从 Controller 重定向到 Controller ，不能到 JSP。

```java
@RequestMapping("/a")
public String a(RedirectAttributes ra){
    ra.addAttribute("msg","普通方式");
    ra.addFlashAttribute("msg2", "msg2");
    return "redirect:/b";
}

@RequestMapping("/b")
public ModelAndView b(String msg, @ModelAttribute("msg2") String msg2){
    System.out.println(msg);
    System.out.println(msg2);
    return null;
}
/* 
 * 重定向的地方 a 需要一个RedirectAttributes参数，当使用 addAttribute() 方法时需要在目标地使用设置的相同名  称
 * 如果使用 addFlashAttribute()方法，则需要在目标地方法参数中添加 @ModelAttribute("")注解，name值和设置的值一样。
 */
```

- **参数处理：** 

  获取请求参数，需要保证请求参数名称和 Controller 方法的形式参数名称一致，如果请求参数名称和形参名称不同需要添加一个 `@RequestParam` 注解，设置传递过来的参数名称。

  **编码设置：**

  ```xml
  <!--3、字符编码过滤器，一定要放在所有过滤器之前-->
  <filter>
      <filter-name>characterEncoding</filter-name>
      <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
      <init-param>
          <param-name>encoding</param-name>
          <param-value>UTF-8</param-value>
      </init-param>
      <!--强制使用字符编码-->
      <init-param>
          <param-name>forceRequestEncoding</param-name>
          <param-value>true</param-value>
      </init-param>
      <init-param>
          <param-name>forceResponseEncoding</param-name>
          <param-value>true</param-value>
      </init-param>
  </filter>
  <filter-mapping>
      <filter-name>characterEncoding</filter-name>
      <url-pattern>/*</url-pattern>
  </filter-mapping>
  ```

- **ModelAttribute注解：** 

  1、给共享的 model 数据设置 key 名，可以添加在形参上，也可以添加在方法上。针对符合类型的参数，缺省情况下会方法 model 中做数据共享，缺省的 key 就是类型的首字母小写。

  ```java
  @RequestMapping("/b")
  public String b(User u){
      return "result";
      //Controller 跳转到 result.jsp 页面，此时在jsp页面上可以通过 ${user} 得到共享数据。
  }
  
  @RequestMapping("/c")
  public String c(@ModelAttribute("myUser") User u){
      return "result";
      //使用 @ModelAttribute("myUser") 注解后在 jsp 页面就需要使用 ${myUser} 获取共享数据。
  }
  ```

  2、可以标注一个非请求处理的方法，被标注的方法，每次在请求处理之前都会优先被调用。一般不使用。

- **获取其它请求信息：** 

  `@RequestHeader("")` 获取请求头信息，可以写入请求头中的名称来获取指定的值。

  `@CookieValue("")` 通过 Cookie 名称，获取指定的 Cookie 值。

  `@SessionAttributes("")` 将指定 key 的 model 数据存放到 session 中。

#### 10、使用 Jackson 处理 JSON 数据：

`@RequestBody` 处理请求，用于读取 Http 请求的内容（JSON格式字符串），转换为 Java 对象

`@ResponseBody` 处理相应，用于将 Controller 的方法返回的对象转换成 JSON 字符串，添加在方法上，对当前方法做 JSON 处理，添加到类上会对当前类中的所有方法做 JSON 处理。

`@RestController` @Controller + @ResponseBody

JSON 相应乱码问题：@RequestMapping(value = "/dept", produces = MediaType.APPLICATION_JSON_UTF8_VALUE) 在注解中添加 produces = MediaType.APPLICATION_JSON_UTF8_VALUE。

#### 11、日期类型处理：

**后台接收前台时间类型数据：**

```java
@RequestMapping("/date")
public ModelAndView date(@DateTimeFormat(pattern = "yyyy-MM-dd") Date date){
    return null;
}
```

如果是对象中的时间属性，只需要在属性上添加 `@DateTimeFormat(pattern = "yyyy-MM-dd")` 注解即可。

**后台向前提响应 JSON 格式的时间类型：**

```java
public class User{
    private String name;
    private Integer age;
    @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss",timezone="GMT+8")
    private Date birthday;
}
```

当后台向前台响应对象时，对象中的 Date 属性的值会变成毫秒值，此时只要在对象中的Date属性上添加 `@JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss",timezone="GMT+8")` 注解即可。

#### 12、SpringMVC工具类（RequestContextHolder）：

能够在任何地方获取 Request 对象。

需求：当用户登录后需要将用户的登录信息保存到当前的 Session 域中。

```java
public class UserContext{
    
    private static final String USER_IN_SESSION = "user_in_session";
    
    //获取 HttpSession 对象
    public static HttpSession getSession(){
        return ((ServletRequestAttributes) (RequestContextHolder.getRequestAttributes())).getRequest().getSession();
    }

    //向session中设置当前用户信息
    public static void setCurrentUser(User current){
        if (current == null) {//如果用户信息为空则清空session域中数据
            getSession().invalidate();
        } else {
            getSession().setAttribute(USER_IN_SESSION, current);
        }
    }
    
    //获取session中的当前对象
    public static User getCurrentUser(){
        return (User) getSession().getAttribute(USER_IN_SESSION);
    }
}
```

#### 13、拦截器	

![](G:\云汇科技\笔记\浙思云汇\assets/拦截器执行过程.png)

```java
public class myInterceptor implements HandlerInterceptor {

    /**
     * 请求到达 Handler 之前会先执行这个前置方法，当方法返回 false 时请求直接返回，不会进入下一个拦截器
     * 只有返回 true 时才会向链中下一个处理节点传递
     */
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        return false;
    }

    /**
     * 控制器方法执行后，视图渲染之前执行
     */
    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {

    }

    /**
     * 视图渲染之后执行
     * 处理Controller异常信息，记录操作日志，清理资源
     */
    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {

    }
}
```

在 xml 中配置声明拦截器：

```xml
<mvc:interceptors>
    <mvc:interceptor>
        <!--对哪些资源做拦截
            /* 只能拦截一级路径
            /** 可以拦截一级或多级路径
        -->
        <mvc:mapping path="/**"/>
        <!--排除不需要被拦截的资源-->
        <mvc:exclude-mapping path="/login"/>
        <!--拦截器类-->
        <bean class="com.yunhui.myInterceptor"/>
    </mvc:interceptor>
</mvc:interceptors>
```

#### 14、异常处理：

全局异常处理，只要出错就统一处理。

```xml
<!--基于 XML 文件的配置形式-->
<bean class="org.springframework.web.servlet.handler.SimpleMappingExceptionResolver">
    <!--配置默认的视图-->
    <property name="defaultErrorView" value="error.jsp"/>
    <!--在错误页面获取异常信息对象变量名称，缺省exception-->
    <property name="exceptionAttribute" value="ex"/>
</bean>
```

基于注解的异常处理：

```java
@ControllerAdvice
public class ExceptionAdvice {
    @ExceptionHandler
    public String error(Exception ex, Model model){
        model.addAttribute("errorMsg", ex.getMessage());
        return "error.jsp";
    }
}
```

#### 15、数据校验：

JSR303标准的校验框架：Spring 在进行数据绑定时，可同时调用数据校验框架完成数据校验工作。

JSR303 是一个数据校验标准，其最终是由 Hibernate validator 做具体的实现。

|     属性     |                  描述                  |
| :----------: | :------------------------------------: |
|   @NotNull   |           注解元素必须是非空           |
|    @Null     |            注解元素必须是空            |
|   @Digits    |          验证数字构成是否合法          |
|   @Future    |       验证是否在当前系统时间之后       |
|    @Past     |       验证是否在当前系统时间之前       |
|     @Max     |    验证值是否小于等于最大指定整数值    |
|     @Min     |    验证值是否大于等于最小指定整数值    |
|   @Pattern   |   验证字符串是否匹配指定的正则表达式   |
|    @Size     |      验证元素大小是否在指定范围内      |
| @DecimalMax  |    验证值是否小于等于最大指定小数值    |
| @DecimalMin  |    验证值是否大于等于最小指定小数值    |
| @AssertTrue  |         被注释的元素必须为true         |
| @AssertFalse |        被注释的元素必须为false         |
|    @Email    |     被注释的元素必须是电子邮箱地址     |
|   @Length    | 被注释的字符串的大小必须在指定的范围内 |
|  @NotEmpty   |        被注释的字符串的必须非空        |
|    @Range    |     被注释的元素必须在合适的范围内     |

```java
import javax.validation.constraints.Email;
import javax.validation.constraints.Pattern;
import java.io.Serializable;

public class Employee implements Serializable {
    private Integer empId;

    @Pattern(regexp = "(^[a-zA-Z0-9_-]{6,16}$)|(^[\\u2E80-\\u9FFF]{2,5}$)",
            message = "用户名可以是2~5位中文或者6~16位英文和数字的组合")
    private String empName;
    
	@NotNull(message = "性别不能为空")
    private String gender;
    
    @Email(message = "邮箱格式不正确")
    private String email;
    
	@Size(min = 1, max = 130)
    private Integer dId;
}
```

在实体类中标注相应的注解对属性进行数据格式的限定。然后在 Controller 类中对象填充的时候添加注解 @Valid 标注。

```java
@RequestMapping(value = "/emp", method = RequestMethod.POST)
@ResponseBody
public Msg saveEmp(@Valid Employee employee, BindingResult result) {
    if (result.hasErrors()){
        List<FieldError> errors = result.getFieldErrors();
        Map<String, Object> map = new HashMap<>();
        for (FieldError error : errors) {
            System.out.println("错误的字段名：" + error.getField());
            System.out.println("错误的信息：" + error.getDefaultMessage());
            map.put(error.getField(), error.getDefaultMessage());
        }
        return Msg.fail().add("errorField", map);
    } else {
        employeeService.save(employee);
        return Msg.success();
    }
}
```

#### 16、SpringMVC表单标签：

通过SpringMVC的表单标签可以实现将模型数据中的属性和 HTML 表单元素相绑定，以实现表单数据更便捷编辑和表单值的回显。

```jsp
<%@ taglib uri="http://www.springframework.org/tags/form" prefix="form"%>
```

|     标签     |                  描述                   |
| :----------: | :-------------------------------------: |
|     form     |              渲染表单元素               |
|    input     |     渲染\<input type=“text”/> 元素      |
|   password   |   渲染\<input type=“password”/> 元素    |
|    hidden    |    渲染\<input type=“hidden”/> 元素     |
|   textarea   |           渲染 textarea 元素            |
|   checkbox   | 渲染一个 \<input type=“checkbox”/> 元素 |
|  checkboxes  | 渲染多个 \<input type=“checkbox”/> 元素 |
| radiobutton  |  渲染一个 \<input type=“radio”/> 元素   |
| radiobuttons |  渲染多个 \<input type=“radio”/> 元素   |
|    select    |            渲染一个选择元素             |
|    option    |            渲染一个可选元素             |
|   options    |          渲染一个可选元素列表           |
|    errors    |         在 span 中渲染字段错误          |

#### 17、SpringMVC文件上传和下载：

- **基于 Apache 的上传组件：**

  配置文件上传解析器：

  ```xml
  <!-- bean 的id必须为 multipartResolver -->
  <bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
      <!--限制上传文件的大小-->
      <property name="maxUploadSize" value="1048576"/>
  </bean>
  ```

  文件上传代码：

  ```java
  @Controller
  public class FileUploadController {
  
      @Autowired
      private ServletContext servletContext;
  
      @RequestMapping("/file_on")
      @ResponseBody
      //注意：multipartFile 属性名必须和页面上文件上传控件的 name 值一致
      public void getFile(User user, MultipartFile multipartFile) throws IOException {
          String fileName = multipartFile.getOriginalFilename();
          String saveDir = servletContext.getRealPath("/upload");
          Files.copy(multipartFile.getInputStream(), Paths.get(saveDir, fileName));
      }
  }
  ```

- **基于 Servlet3.0 的上传：**

  配置文件上传解析器：

  ```xml
  <!-- Servlet3.0 的文件上传解析器-->
  <bean id="multipartResolver" class="org.springframework.web.multipart.support.StandardServletMultipartResolver"/>
  ```

  在 web.xml 中对文件上传的属性进行配置：

  ```xml
  <servlet>
      <servlet-name>dispatcherServlet</servlet-name>
      <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
      <load-on-startup>1</load-on-startup>
      <!--文件上传的配置-->
      <multipart-config>
          <!--本地文件存储位置-->
          <location>C:/temp</location>
          <!--上传文件的最大容量-->
          <max-file-size>1000000</max-file-size>
          <!--一次请求最大上传量-->
          <max-request-size>1000000</max-request-size>
          <!--超过10KB就存到硬盘中-->
          <file-size-threshold>10240</file-size-threshold>
      </multipart-config>
  </servlet>
  ```

- **文件下载：**

  使用原始 Servlet 的文件下载：

  ```java
  @Controller
  public class FileDownController {
  
      @RequestMapping("/down")
      public void download(String fileName, HttpServletRequest request, HttpServletResponse response) throws IOException {
          String dir = request.getServletContext().getRealPath("/WEB-INF/down");
          //设置响应头：下载文件
          response.setContentType(MediaType.APPLICATION_OCTET_STREAM_VALUE);
          //设置建议保存名称
          response.setHeader("Content-Disposition","attachment;fileName=" + fileName);
          Files.copy(Paths.get(dir, fileName), response.getOutputStream());
      }
  }
  ```

  使用 SpringMVC 的方式下载：

  ```java
  @RequestMapping("/down2")
  public ResponseEntity<byte[]> download2(String fileName, HttpServletRequest request, HttpServletResponse response) throws IOException {
      String dir = request.getServletContext().getRealPath("/WEB-INF/down");
      HttpHeaders headers = new HttpHeaders();
      //设置响应头：下载文件
      headers.setContentType(MediaType.APPLICATION_OCTET_STREAM);
      //设置建议保存名称
      headers.setContentDispositionFormData("attachment", fileName);
      return new ResponseEntity<>(FileUtils.readFileToByteArray(new File(dir, fileName)), headers, HttpStatus.CREATED);
  }
  ```


**解决 @ResponseBody 返回乱码问题：** 

```xml
<mvc:annotation-driven>
    <!--解决 JSON 返回乱码-->
    <mvc:message-converters register-defaults="true">
        <bean class="org.springframework.http.converter.StringHttpMessageConverter">
            <property name="supportedMediaTypes">
                <list>
                    <value>text/html; charset=UTF-8</value>
                    <value>application/json; charset=UTF-8</value>
                </list>
            </property>
        </bean>
    </mvc:message-converters>
</mvc:annotation-driven>
```

> 注意： 该配置需要写在注解扫描上面。

#### 18、SSM整合配置：

SpringMVC配置文件：

```xml
<!--spring-mvc配置文件，包含网站跳转逻辑的控制、配置-->
<context:component-scan base-package="com.yunhui" use-default-filters="false">
    <!--只扫描控制器-->
    <context:include-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
</context:component-scan>

<!--配置视图解析器，方便页面返回-->
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    <property name="prefix" value="/WEB-INF/views/"/>
    <property name="suffix" value=".jsp"/>
</bean>

<!--两个标准配置-->
<!--将spring-mvc不能处理的请求交给tomcat-->
<mvc:default-servlet-handler/>
<!--能支持spring-mvc更高级的功能，JSR303校验，Ajax请求，映射动态请求-->
<mvc:annotation-driven/>
```

Spring配置文件整合：

```xml
<!--Spring主配置文件-->
<context:component-scan base-package="com.ssm">
    <!--扫描注解但不扫描 Controller 注解-->
    <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
</context:component-scan>

<!--配置Mybatis整合-->
<!--加载外部配置文件-->
<context:property-placeholder location="classpath:db.properties"/>

<!--配置数据源c3p0-->
<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
    <property name="driverClass" value="${jdbc.driver}"/>
    <property name="jdbcUrl" value="${jdbc.url}"/>
    <property name="user" value="${jdbc.username}"/>
    <property name="password" value="${jdbc.password}"/>
</bean>

<!--配置Mybatis整合-->
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
    <property name="configLocation" value="classpath:mybatis-config.xml"/>
    <property name="dataSource" ref="dataSource"/>
    <property name="mapperLocations" value="classpath:mappers/*.xml"/>
</bean>

<!--配置Mybatis的映射对应的接口-->
<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
    <property name="basePackage" value=""/>
</bean>

<!--配置事物管理-->
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource"/>
</bean>
```

