## Shiro安全框架

Shiro 是 Java 的一个安全（权限框架），Shiro 可以完成认证、授权、加密、会话管理、与WEB集成、缓存等。

**Principal：** 身份信息，是主体进行身份认证的标识，标识具有唯一性，一个主体可以有多个身份，但必须有一个主身份（Primary Principal）。

**credential：** 凭证信息，是只有主体自己知道的安全信息，如密码、证书等。

#### 一、功能介绍：

|      功能       |                         说明                         |
| :-------------: | :--------------------------------------------------: |
| Authentication  |                    身份认证/登录                     |
|  Authorization  |       授权，验证某个已认证的用户是否有某个权限       |
| Session Manager | 会话管理，用户登录后在没有退出前所有的信息都在会话中 |
|  Cryptography   |                         加密                         |
|   Web Support   |                       web 支持                       |
|     Caching     |                         缓存                         |
|   Concurrency   |               支持多线程应用的并发验证               |
|     Testing     |                     提供测试支持                     |
|     Run As      |      允许一个用户假装成另一个用户的身份进行访问      |
|   Remember Me   |                        记住我                        |

**Subject：** 主体。应用代码直接交互的对象是 Subject ，Subject 代表了当前用户，与当前交互的任何东西都是 Subject ，与 Subject 的所有交互都会委托给 SecurityManager。Subject 其实是一个门面，SecurityManager 才是实际的执行者。

**SecurityManager：** 安全管理器，所有与安全相关的操作都会与 SecurityManager 交互，且其管理着所有的 Subject ，SecurityManager 是 Shiro 的核心，负责与 Shiro 的其它组件进行交互。

**Realm：** Shiro 从 Realm 获取安全数据（如用户、角色、权限），可以把 Realm 看做是 DataSource。

#### 二、简单案例：

1、拷贝依赖；

2、配置 shiro.ini 文件

3、执行 shiro 登录，登出操作

```ini
#配置 shiro.ini
#用户---> 用户名 = 密码, 角色1, 角色2...
[users]
# user 'root' with password 'secret' and the 'admin' role
root = secret, admin
# user 'guest' with the password 'guest' and the 'guest' role
guest = guest, guest
# user 'presidentskroob' with password '12345' ("That's the same combination on
# my luggage!!!" ;)), and role 'president'
presidentskroob = 12345, president
# user 'darkhelmet' with password 'ludicrousspeed' and roles 'darklord' and 'schwartz'
darkhelmet = ludicrousspeed, darklord, schwartz
# user 'lonestarr' with password 'vespa' and roles 'goodguy' and 'schwartz'
lonestarr = vespa, goodguy, schwartz

#角色： 角色 = 行为
[roles]
# 'admin' role has all permissions, indicated by the wildcard '*'
admin = *
# The 'schwartz' role can do anything (*) with any lightsaber:
schwartz = lightsaber:*
# The 'goodguy' role is allowed to 'drive' (action) the winnebago (type) with
# license plate 'eagle5' (instance specific id)
# 具体行为：角色 = 类型:行为:实例 ---> user:delete:zhangsan 用户user可以删除zhangsan的信息
goodguy = winnebago:drive:eagle5
```

```java
public void test1(){
    // 构建SecurityManager工厂，IniSecurityManagerFactory可以从ini文件中初始化SecurityManager环境
    IniSecurityManagerFactory factory = new IniSecurityManagerFactory("classpath:shiro.ini");
    // 通过工厂创建SecurityManager
    SecurityManager securityManager = factory.getInstance();
	// 将securityManager设置到运行环境中
    SecurityUtils.setSecurityManager(securityManager);

    // 创建一个Subject实例，该实例认证要使用上边创建的securityManager进行
    Subject subject = SecurityUtils.getSubject();
    // 创建token令牌，记录用户认证的身份和凭证即账号和密码
    UsernamePasswordToken token = new UsernamePasswordToken("lonestarr", "vespa");
    try {
        // 用户登陆
        subject.login(token);
    } catch (AuthenticationException e) {
        // TODO Auto-generated catch block
        e.printStackTrace();
    }
    // 用户认证状态
    boolean isAuthenticated = subject.isAuthenticated();
    System.out.println("用户认证状态：" + isAuthenticated);
    //测试角色
    if (subject.hasRole("schwartz")) {
        System.out.println("May the Schwartz be with you!");
    } else {
        System.out.println("Hello, mere mortal.");
    }
    //是否具备某一个行为
    if (subject.isPermitted("lightsaber:wield")) {
        System.out.println("You may use a lightsaber ring.  Use it wisely.");
    } else {
        System.out.println("Sorry, lightsaber rings are for schwartz masters only.");
    }
    //是否具备某一具体行为
    if (subject.isPermitted("winnebago:drive:eagle5")) {
       System.out.println("You are permitted to 'drive' the winnebago with license plate (id) 'eagle5'.  " + "Here are the keys - have fun!");
    } else {
        System.out.println("Sorry, you aren't allowed to drive the 'eagle5' winnebago!");
    }
    // 用户退出
    subject.logout();
    isAuthenticated = subject.isAuthenticated();
    System.out.println("用户认证状态：" + isAuthenticated);
}
```

> 构造 SecurityManager 环境 ---> Subject.login()提交认证 ---> SecurityManager.login()执行认证 ---> Authenticator执行认证（认证器，最终执行认证） ---> Realm 根据身份获取认证信息
>
> 认证执行过程：
>
> 1、通过 token 获取 username，接着去查找 realm 中是否存在 username 用户，如果有将用户数据包装成 AuthenticationInfo 对象并返回，如果没有则返回 null。
>
> 2、比对 token 中密码跟 info 中的密码，如果一致表示登录成功，否则登录失败。

#### 三、自定义 realm 登录登出：

ini 配置文件的弊端：不够灵活，密码不能加密。

1、自定义 realm 类并集成 AuthorizingRealm 类。重写3个方法： `getName()` `doGetAuthorizationInfo` `doGetAuthenticationInfo` 

```java
public class MyRealm extends AuthorizingRealm {

    @Override
    public String getName() {
        return "MyRealm";
    }

    //授权操作
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
        return null;
    }

    //认证操作
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {
        //通过用户名去数据库中查询用户信息，封装成一个 AuthenticationInfo 对象，方便验证器对比
        //authenticationToken：表示登录时包装的 UsernamePasswordToken 对象
        //获取 token 中的用户名
        String username = (String) authenticationToken.getPrincipal();

        //通过用户名查询数据库，将该用户对应的数据查询出来：账号、密码
        if (!"zhangsan".equals(username)){
            return null;
        }

        String password = "666";

        //SimpleAuthenticationInfo 对象表示realm登录比对信息，
        // 参数1：用户信息（实际登录中是用户真实对象）
        // 参数2：密码
        // 参数3：当前realm的名字
        return new SimpleAuthenticationInfo(username, password, getName());
    }
}
```

2、配置 ini 文件，指定使用自定义的 realm 

```ini
#自定义 realm
myRealm= com.yunhui.shiro.MyRealm
#指定 securityManager 的 realm 实现
securityManager.realms=$myRealm
```

3、加载配置文件，执行登录操作。

#### 四、Shiro 加密操作：

散列算法一般用于生成数据的摘要信息，是一种不可逆的算法，一般适用于存储密码之类的数据，常见的散列算法有MD5、SHA等。一般进行散列时最好提供一个 salt（盐），如果直接对密码进行散列相对来说更容易破解。

```java
@Test
public void testMD5(){
    String password = "666"; //密码，明文

    //MD5加密
    Md5Hash md5Hash = new Md5Hash(password);
    //fae0b27c451c728867a567e8c1bb4e53
    System.out.println(md5Hash.toString());

    //加密：MD5 + 盐
    md5Hash = new Md5Hash(password, "zhangsan");
    //2f1f526e25fdefa341c7a302b47dd9df
    System.out.println(md5Hash);

    //加密：MD5 + 盐 + 散列次数
    md5Hash = new Md5Hash(password, "zhangsan",3);
    //cd757bae8bd31da92c6b14c235668091
    System.out.println(md5Hash);
}
```

使用加密后验证密码（加盐）：

1、自定义加密之后的 realm 继承 AuthorizingRealm 类，重写3个方法；

```java
public class PasswordRealm extends AuthorizingRealm {

    @Override
    public String getName() {
        return "PasswordRealm";
    }

    //授权
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
        return null;
    }

    //认证
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {
        //通过用户名去数据库中查询用户信息，封装成一个 AuthenticationInfo 对象，方便验证器对比
        //authenticationToken：表示登录时包装的 UsernamePasswordToken 对象
        //获取 token 中的用户名
        String username = (String) authenticationToken.getPrincipal();

        //通过用户名查询数据库，将该用户对应的数据查询出来：账号、密码
        if (!"zhangsan".equals(username)){
            return null;
        }

        //模拟数据库中保存的加密之后的密文：666 + 账号 + 盐 + 散列次数3次
        String password = "cd757bae8bd31da92c6b14c235668091";

        //SimpleAuthenticationInfo 对象表示realm登录比对信息，
        // 参数1：用户信息（实际登录中是用户真实对象）
        // 参数2：密码
        // 参数3：盐
        // 参数4：当前realm的名字
        return new SimpleAuthenticationInfo(username, password, ByteSource.Util.bytes("zhangsan"), getName());
    }
}
```

2、配置 ini 文件

```ini
[main]
#定义凭证匹配器
credentialsMatcher= org.apache.shiro.authc.credential.HashedCredentialsMatcher
#散列算法
credentialsMatcher.hashAlgorithmName=md5
#散列次数
credentialsMatcher.hashIterations=3

#将凭证匹配器设置到realm
myRealm=com.yunhui.shiro.PasswordRealm
myRealm.credentialsMatcher=$credentialsMatcher
securityManager.realms=$myRealm
```

3、加载配置文件测试。

#### 五、Shiro 授权：

**RBAC 授权模式：** 基于角色的权限管理，简单理解为，谁扮演什么角色，被允许做什么操作。

- 用户对象：user，当前操作用户
- 角色对象：role，表示权限操作许可权的集合
- 权限对象：permission，资源操作许可权

**授权方式：** 

- 编程式： 通过 if/else 授权代码完成

    ```java
    Subject subject = SecurityUtils.getSubject();
    if (subject.hasRole("admin")){
        //有权限
    } else {
        //无权限
    }
    ```

    ```ini
    #用户---> 用户名 = 密码, 角色1, 角色2...
    [users]
    # user 'root' with password 'secret' and the 'admin' role
    root = secret, admin
    # user 'guest' with the password 'guest' and the 'guest' role
    guest = guest, guest
    
    #角色： 角色 = 行为
    [roles]
    # 'admin' role has all permissions, indicated by the wildcard '*'
    admin = *
    # The 'schwartz' role can do anything (*) with any lightsaber:
    schwartz = lightsaber:*
    # The 'goodguy' role is allowed to 'drive' (action) the winnebago (type) with
    # license plate 'eagle5' (instance specific id)
    # 具体行为：角色 = 类型:行为:实例 ---> user:delete:zhangsan 用户user可以删除zhangsan的信息
    goodguy = winnebago:drive:eagle5
    ```

    > 在 ini 文件中用户、角色、权限的配置规则是：用户名=密码，角色1，角色2        角色=权限1，权限2；首先根据用户名找角色，再根据角色找权限，角色是权限的集合。
    >
    > 权限字符串的规则是： 资源标识符：操作：资源实例标识符；即对哪个资源的哪个实例具有什么操作，权限字符串也可以使用 * 通配符。
    >
    > 用户修改实例001的权限：user:update:001

- 注解方式： 通过在执行的 java 方法上放置相应的注解完成

    ```java
    @RequiresRoles("admin")
    public void hello(){
        //有权限执行
    }
    ```

- JSP 标签方式： 在 JSP 页面通过相应的标签完成

    ```jsp
    <shiro:hasRole name="admin">
    	<!--有权限-->
    </shiro:hasRole>
    ```

验证用户是否拥有权限： subject.isPermitted("user:delete");

**自定义 Realm 进行授权：** 

1、创建自定义 realm 类并继承 AuthorizingRealm 类。

```java
public class PermissionRealm extends AuthorizingRealm {
    //授权
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {

        //传入参数：principalCollection 用户认证的凭证信息
        //SimpleAuthenticationInfo：认证方法返回的封装认证信息中第一个参数：用户信息（username）
        //当前登录的用户信息：用户凭证
        String username = (String) principalCollection.getPrimaryPrincipal();
        //模拟查询数据库：查询用户事先指定的角色，以及用户权限
        List<String> roles = new ArrayList<>();
        List<String> permission = new ArrayList<>();
        //假设用户在数据库中有 role1 的角色
        roles.add("role1");
        //假设用户在数据库中有 user：delete 权限
        permission.add("user:delete");
        //返回用户在数据库中的权限与角色
        SimpleAuthorizationInfo info = new SimpleAuthorizationInfo();
        info.addRoles(roles);
        info.addStringPermissions(permission);
        return info;
    }

    //认证
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {
        //通过用户名去数据库中查询用户信息，封装成一个 AuthenticationInfo 对象，方便验证器对比
        //authenticationToken：表示登录时包装的 UsernamePasswordToken 对象
        //获取 token 中的用户名
        String username = (String) authenticationToken.getPrincipal();

        //通过用户名查询数据库，将该用户对应的数据查询出来：账号、密码
        if (!"zhangsan".equals(username)){
            return null;
        }
        //模拟数据库中保存的加密之后的密文：666 + 账号 + 盐 + 散列次数3次
        String password = "cd757bae8bd31da92c6b14c235668091";
        //SimpleAuthenticationInfo 对象表示realm登录比对信息，
        // 参数1：用户信息（实际登录中是用户真实对象）
        // 参数2：密码
        // 参数3：盐
        // 参数4：当前realm的名字
        return new SimpleAuthenticationInfo(username, password, ByteSource.Util.bytes("zhangsan"), getName());
    }
}
```

2、配置 ini 文件：

```ini
[main]
#定义凭证匹配器
credentialsMatcher= org.apache.shiro.authc.credential.HashedCredentialsMatcher
#散列算法
credentialsMatcher.hashAlgorithmName=md5
#散列次数
credentialsMatcher.hashIterations=3

#自定义 realm
myRealm=com.yunhui.shiro.PermissionRealm
myRealm.credentialsMatcher=$credentialsMatcher
#指定 securityManager 的 realm 实现
securityManager.realms=$myRealm
```

3、测试验证

> 权限验证流程：
>
> 构造 SecurityManager 环境 -----> Subject.isPermitted()授权 -----> SecurityManager.isPermitted()执行授权 -----> Authorizer执行授权 -----> Realm 根据身份获取权限信息

#### 六、web 集成：

Shiro 与 web 集成，主要是通过配置一个 `ShiroFilter` 拦截所有URL，其中 ShiroFilter 类似于 SpringMVC 的前端控制器。是所有请求的入口点，负责根据配置文件，判断请求进入URL是否需要登录/授权等操作。

1、添加相关依赖：

```xml
<dependency>
    <groupId>commons-logging</groupId>
    <artifactId>commons-logging</artifactId>
    <version>1.2</version>
</dependency>
<!--shiro的核心依赖-->
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-core</artifactId>
    <version>1.4.0</version>
</dependency>
<!--shiro对web的支持-->
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-web</artifactId>
    <version>1.4.0</version>
</dependency>
<!--Servlet-->
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>javax.servlet-api</artifactId>
    <version>4.0.1</version>
    <scope>provided</scope>
</dependency>
```

2、配置 shiroFilter 过滤器拦截所有请求：

```xml
<context-param>
    <param-name>shiroEnvironmentClass</param-name>
    <param-value>org.apache.shiro.web.env.IniWebEnvironment</param-value>
</context-param>
<context-param>
    <param-name>shiroConfigLocations</param-name>
    <param-value>classpath:shiro.ini</param-value>
</context-param>
    
<listener>
    <listener-class>org.apache.shiro.web.env.EnvironmentLoaderListener</listener-class>
</listener>

<!--配置 shiroFilter 过滤器拦截所有请求-->
<filter>
    <filter-name>shiroFilter</filter-name>
    <filter-class>org.apache.shiro.web.servlet.ShiroFilter</filter-class>
</filter>
<filter-mapping>
    <filter-name>shiroFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

3、配置 ini 文件：

```ini
[main]
#默认是/login.jsp
authc.loginUrl=/login.jsp
#用户没有需要的角色时跳转的页面
roles.unauthorizedUrl=/nopermission.jsp
#用户没有需要的权限时跳转的页面
perms.unauthorizedUrl=/nopermission.jsp
#登出之后重定向的页面
logout.redirectUrl=/main

[users]
admin=666,admin
wf=666,deptMgr

[roles]
admin=employee:*,department:*
deptMgr=department:view

[urls] #配置URL权限控制， url路径=shiro默认的权限拦截过滤器
#静态资源可以匿名访问
/static/**=anon
#访问员工列表需要身份认证及需要拥有admin角色   authc 登录验证
/employee=authc,roles[admin]
#访问部门列表需要身份认证及需要拥有department:view权限
/department=authc,perms["department:view"]
#请求 logOut 会被logout捕获并清除 session
/logOut=logout
#所有的请求都需要身份认证
/**=authc
```

| 过滤器简称 |               说明               |
| :--------: | :------------------------------: |
|    anon    | 匿名拦截器，即不需要登录即可访问 |
|   authc    |   表示需要认证（登录）才能使用   |
| authcBasic |     Basic HTTP身份验证拦截器     |
|   roles    |          角色授权拦截器          |
|   perms    |          权限授权拦截器          |
|    user    |            用户拦截器            |
|   logout   |            退出拦截器            |
|    port    |            端口拦截器            |
|    rest    |         rest 风格拦截器          |
|    ssl     |            SSL 拦截器            |

**authc 拦截器工作原理：** 

第一次发送请求（/main），authc 拦截器拦截请求，发现当前用户没有登录，直接跳转到 `authc.loginUrl` 路径，同时保存当前请求路径。

authc 拦截器的作用：

1、登录认证： 请求进来时，拦截并判断当前用户是否登录，如果已经登录则放行，如果没有登录，跳转到 authc.loginUrl 属性配置的路径，默认是 login.jsp。

2、执行登录认证： 请求进来时，如果请求的路径为 anthc.loginUrl 属性配置的路径（没配置，默认是/login.jsp）时，如果当前用户没有登录，authc 拦截器会尝试获取请求中的账号和密码值，然后比对 ini 配置文件或者 realm 中的用户列表，如果比对正确，直接执行登录操作，反之抛出异常，跳转到 authc.loginUrl 路径。注意：请求中账号和密码必须固定为 username 跟 password，如果需要改动必须额外指定，authc.usernameParam=xxx      authc.passwordParam=xxx。

#### 七、Spring 集成 Shiro：

1、添加 shiro 与 spring 集成依赖 jar 包：

```xml
<!--shiro-->
<dependency>
    <groupId>commons-logging</groupId>
    <artifactId>commons-logging</artifactId>
    <version>1.1.3</version>
</dependency>
<dependency>
    <groupId>commons-collections</groupId>
    <artifactId>commons-collections</artifactId>
    <version>3.2.1</version>
</dependency>

<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-core</artifactId>
    <version>1.4.0</version>
</dependency>

<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-web</artifactId>
    <version>1.4.0</version>
</dependency>
<dependency>
    <groupId>net.sf.ehcache</groupId>
    <artifactId>ehcache-core</artifactId>
    <version>2.6.8</version>
</dependency>
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-ehcache</artifactId>
    <version>1.4.0</version>
</dependency>

<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-quartz</artifactId>
    <version>1.4.0</version>
</dependency>

<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-spring</artifactId>
    <version>1.4.0</version>
</dependency>
```

2、在 web.xml 中配置 shiro 与 spring 框架集成需要的 shiroFilter 代理类

```xml
<!-- shiro过虑器，DelegatingFilterProx会从spring容器中找shiroFilter -->
<filter>
    <filter-name>shiroFilter</filter-name>
    <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
    <init-param>
        <param-name>targetFilterLifecycle</param-name>
        <param-value>true</param-value>
    </init-param>
</filter>
<filter-mapping>
    <filter-name>shiroFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

3、配置 spring-shiro 配置文件：

```xml
<!--1、配置自定义的realm-->
<bean id="userRealm" class="com.yunhui.spring.shiro.realm.UserRealm">
	<!--密码需要加密：加密器-->
	<property name="credentialsMatcher" ref="credentialsMatcher" />
	<property name="userDAO" ref="userDAOImpl"/>
	<property name="roleDAO" ref="roleDAOImpl"/>
	<property name="permissionDAO" ref="permissionDAOImpl"/>
</bean>

<!-- 2、配置安全管理器SecurityManager -->
<bean id="securityManager" class="org.apache.shiro.web.mgt.DefaultWebSecurityManager">
	<property name="realm" ref="userRealm"/>
	<!--给shiro添加缓存机制-->
	<property name="cacheManager" ref="cacheManager"/>
</bean>

<!-- 3、定义ShiroFilter  id必须和 web.xml 中配置的代理 filter 一致-->
<bean id="shiroFilter" class="org.apache.shiro.spring.web.ShiroFilterFactoryBean">
	<property name="securityManager" ref="securityManager"/>
	<property name="loginUrl" value="/login"/>
	<property name="unauthorizedUrl" value="/nopermission.jsp"/>
	<property name="filterChainDefinitions">
		<value>
			/logout=logout
			/**=authc
		</value>
	</property>
</bean>
```

spring-shiro.xml 文件配置完成后，在 mvc.xml 主配置文件中引入。

**使用注解开发：** 

1、在 spring-shiro 文件中开启 shiro 注解支持：

```xml
<!-- 开启aop，对类代理 -->
<aop:config proxy-target-class="true"/>
<!-- 开启shiro注解支持 -->
<bean class="org.apache.shiro.spring.security.interceptor.AuthorizationAttributeSourceAdvisor">
	<property name="securityManager" ref="securityManager"/>
</bean>
```

2、在方法上添加注解 `@RequiresPermissions("")` ，进行权限控制：

```java
@RequestMapping("/save")
@RequiresPermissions("employee:save")
@PermissionName("员工保存")
public String save() throws  Exception{
    System.out.println("执行了员工保存....");
    return "employee";
}
```

**Shiro 缓存：** 

1、添加相关依赖：

```xml
<dependency>
    <groupId>net.sf.ehcache</groupId>
    <artifactId>ehcache-core</artifactId>
    <version>2.6.8</version>
</dependency>
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-ehcache</artifactId>
    <version>1.4.0</version>
</dependency>
```

2、配置缓存管理器：

```xml
<!-- 缓存管理器开始 -->
<bean id="cacheManager" class="org.apache.shiro.cache.ehcache.EhCacheManager">
	<property name="cacheManager" ref="ehCacheManager"/>
</bean>
<bean id="ehCacheManager" class ="org.springframework.cache.ehcache.EhCacheManagerFactoryBean">
	<property name="configLocation" value="classpath:shiro-ehcache.xml" />
	<property name="shared" value="true"/>
</bean>
```

3、配置缓存配置文件：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ehcache>
	<defaultCache
			maxElementsInMemory="1000"
			eternal="false"
			timeToIdleSeconds="120"
			timeToLiveSeconds="120"
			memoryStoreEvictionPolicy="LRU">
	</defaultCache>
</ehcache>
```

4、在 securityManager 中启用缓存：

```xml
<!-- 2、配置安全管理器SecurityManager -->
<bean id="securityManager" class="org.apache.shiro.web.mgt.DefaultWebSecurityManager">
	<!--使用自定义realm-->
	<property name="realm" ref="userRealm"/>
	<!--给shiro添加缓存机制-->
	<property name="cacheManager" ref="cacheManager"/>
</bean>
```

5、清空缓存设置：手动清除

```java
//清除缓存
public void clearCached() {
    //获取当前等的用户凭证，然后清除
    PrincipalCollection principals = SecurityUtils.getSubject().getPrincipals();
    super.clearCache(principals);
}
```

**加密配置：**

1、在 spring-shiro.xml 文件中配置加密器：

```xml
<!--加密器-->
<bean id="credentialsMatcher" class="org.apache.shiro.authc.credential.HashedCredentialsMatcher">
    <!--加密算法-->
	<property name="hashAlgorithmName" value="md5" />
	<!--散列次数-->
	<property name="hashIterations" value="3" />
</bean>
```

2、在自定义 realm 的 bean 中声明加密器

```xml
<!--1、配置自定义的realm-->
<bean id="userRealm" class="com.yunhui.spring.shiro.realm.UserRealm">
	<!--密码需要加密：加密器-->
	<property name="credentialsMatcher" ref="credentialsMatcher" />
	<property name="userDAO" ref="userDAOImpl"/>
	<property name="roleDAO" ref="roleDAOImpl"/>
	<property name="permissionDAO" ref="permissionDAOImpl"/>
</bean>
```

3、在自定义 realm 中指定加密盐

```java
new SimpleAuthenticationInfo(user, //当前用户对象
            user.getPassword(),//用户密码
            ByteSource.Util.bytes(user.getUsername()),//加密盐值
            getName());//当前 realm 名称
```

