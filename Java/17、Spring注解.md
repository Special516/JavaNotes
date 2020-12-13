## 一、组件注册

#### 1、使用 @Configuration 和 @Bean 给容器中注册组件

```java
/**
 * 配置类 == 配置文件
 * @Configuration 标识这是一个配置类
 */
@Configuration
public class MainConfig {
    /**
     * @Bean 给容器中注册一个 bean
     * 类型为返回值类型，默认使用方法名作为id
     */
    @Bean
    public Person person(){
        return new Person("李四",20);
    }
}
```

```java
/**
 * 测试
 */
public class MainTest {
    public static void main(String[] args) {
        ApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfig.class);
        Person person = (Person) applicationContext.getBean("person");
        System.out.println(person);
    }
}
```

#### 2、@ComponentScan 指定要扫描的包

```java
@ComponentScan(value = "com.wf", excludeFilters = {
		@ComponentScan.Filter(type = FilterType.ANNOTATION, classes = {Controller.class, Service.class})
})
//excludeFilters 指定扫描的时候按照什么规则排除哪些组件
//type = FilterType.ANNOTATION 根据注解过滤

@ComponentScan(value = "com.wf", useDefaultFilters = false, includeFilters = {
		@ComponentScan.Filter(type = FilterType.ANNOTATION, classes = {Controller.class, Service.class}）
})
//includeFilters 指定扫描的时候只扫描什么组件，使用该注解需要设置 useDefaultFilters=false
//FilterType.ANNOTATION：按照注解
//FilterType.ASSIGNABLE_TYPE：按照给定的类型
//FilterType.REGEX：使用正则规则
//FilterType.CUSTOM：使用自定义规则
```

#### 3、@Scope 设置组件作用范围

```java
//@Scope 的常用取值 singleton(单实例)， prototype(多实例)
@Scope("singleton")
@Bean(value = "person1")
public Person person(){
    return new Person("李四",20);
}
```

#### 4、@Lazy 懒加载

```java
//使用 @Lazy 注解实现bean的懒加载，只针对单实例方式，在容器创建的时候并不加载实例
@Lazy
@Bean(value = "person1")
public Person person(){
    return new Person("李四",20);
}
```

#### 5、@Conditional({Condition})  按照条件注册 bean

```java
/**
 * @Conditional({Condition}) 按照一定的条件进行判断，满足条件给容器中注册bean
 */
@Bean("bill")
@Conditional({WindowsCondition.class})//如果是 Windows 系统则创建该实例
public Person person01(){
    return new Person("bill", 60);
}
```

```java
public class WindowsCondition implements Condition {
    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {

        //获取当前环境信息
        Environment environment = context.getEnvironment();

        String property = environment.getProperty("os.name");
        return property.contains("Windows");
    }
}
```

```java
public class LinuxCondition implements Condition {

    /**
     * ConditionContext：判断条件能使用的上下文环境
     * AnnotatedTypeMetadata：注释信息
     * @param context
     * @param metadata
     * @return
     */
    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {

        //1、获取当前 IOC 容器使用的 beanFactory
        ConfigurableListableBeanFactory beanFactory = context.getBeanFactory();
        //2、获取类加载器
        ClassLoader classLoader = context.getClassLoader();
        //3、获取当前环境信息
        Environment environment = context.getEnvironment();
        //4、获取 bean 定义的注册类
        BeanDefinitionRegistry registry = context.getRegistry();

        String property = environment.getProperty("os.name");
        return property.contains("Linux");

    }
}
```

#### 6、@Import 快速的给容器中导入一个组件

```java
//在类上标注 @Import 注解，快速导入组件
@Import(Color.class)
public class MainConfig {}

//com.wf.bean.Color 在容器中 id 默认以全类名加载
```

```java
/**
 * 自定义逻辑，返回需要导入的组件
 */
public class MyImportSelector implements ImportSelector {

    //返回值就是需要导入到组件中的组件的全类名
    //AnnotationMetadata：当前标注 @Import 注解的类的所有注解信息
    @Override
    public String[] selectImports(AnnotationMetadata importingClassMetadata) {
        //方法不要返回 null
        return new String[]{"com.wf.bean.Color"};
    }
}

@Import(value = {MyImportSelector.class})
public class MainConfig {}
```

```java
public class MyImportBeanDefinitionRegistrar implements ImportBeanDefinitionRegistrar {

    /**
     * AnnotationMetadata:当前类的注解信息
     * BeanDefinitionRegistry:Bean定义的注册类
     *      把所有需要添加到容器中的bean通过 BeanDefinitionRegistry.registerBeanDefinition()注册进来
     */
    @Override
    public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
        boolean red = registry.containsBeanDefinition("red");
        if (!red) {
            //指定Bean的定义信息
            RootBeanDefinition rootBeanDefinition = new RootBeanDefinition(Red.class);
            //注册一个bean，指定bean名
            registry.registerBeanDefinition("red", rootBeanDefinition);
        }
    }
}

@Import(value = {MyImportSelector.class, MyImportBeanDefinitionRegistrar.class})
public class MainConfig {}
```

#### 7、FactoryBean

```java
//创建一个 Spring 定义的工厂Bean
public class ColorFactoryBean implements FactoryBean<Color> {

    //返回一个 Color 对象，这个对象会添加到容器中
    @Override
    public Color getObject() throws Exception {
        System.out.println("ColorFactoryBean...getObject...");
        return new Color();
    }

    @Override
    public Class<?> getObjectType() {
        return Color.class;
    }
}
```

```java
@Configuration
public class MainConfig {
	@Bean
    public ColorFactoryBean colorFactoryBean(){
        return new ColorFactoryBean();
    }
}
```

#### 总结：

> 给容器中注册组件：
>     1、包扫描+组件标签注解（@Controller/@Service/@Repository/@Component）
>     2、@Bean[导入第三方包里面的组件]
>     3、@Import[快速给容器中导入一个组件]
>          1)、@Import(要导入到容器中的组件)，容器中会自动注册这个组件，id默认是组件的全类名
>          2)、ImportSelector:返回需要导入的组件的全类名
>          3)、ImportBeanDefinitionRegistrar:手动注册bean到容器中
>     4、使用Spring提供的FactoryBean(工厂Bean)
>          1)、默认获取到的是工厂Bean调用getObject()创建的对象
>          2)、要获取工厂bean本身，需要在id前加一个 "&" 符号

## 二、生命周期

#### 1、@Bean指定初始化和销毁方法

```java
public class Car {
    public Car(){
        System.out.println("car constructor...");
    }
    public void init(){
        System.out.println("car init...");
    }
    public void destroy(){
        System.out.println("car destroy...");
    }
}
```

```java
@Configuration
@ComponentScan("com.wf.bean")
public class MyConfigLifeCycle {
	//通过 @Bean 中的属性设置初始化和销毁的方法
    @Bean(initMethod = "init", destroyMethod = "destroy")
    public Car car(){
        return new Car();
    }
}
```

#### 2、InitializingBean(定义初始化逻辑)，DisposableBean(定义销毁逻辑)

```java
//通过实现 InitializingBean 接口和 DisposableBean 接口，并实现对象的方法，完成初始化和销毁的逻辑
@Component
public class Cat implements InitializingBean, DisposableBean {

    public Cat() {
        System.out.println("Cat constructor...");
    }

    //销毁方法
    @Override
    public void destroy() throws Exception {
        System.out.println("cat destroy");
    }

    //初始化方法
    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("cat afterPropertiesSet");
    }
}
```

#### 3、使用 @PostConstruce(初始化) 注解和 @PreDestroy(销毁) 注解

```java
@Component
public class Dog {
    public Dog(){
        System.out.println("Dog constructor...");
    }

    @PostConstruct
    public void init(){
        System.out.println("Dog postConstruct...");
    }

    @PreDestroy
    public void destroy(){
        System.out.println("Dog preDestroy..");
    }
}
```

#### 4、后置处理器 BeanPostProcessor

```java
/**
 * 后置处理器：初始化前后进行工作
 * 将后置处理器加入到IOC容器中
 * BeanPostProcessor: bean的后置处理器，在bean初始化前后进行一些处理工作
 * 	postProcessBeforeInitialization：在初始化之前工作
 *	postProcessAfterInitialization：在初始化之后工作
 */
@Component
public class MyBeanPostProcessor implements BeanPostProcessor {

    //初始化之前调用
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("postProcessBeforeInitialization..." + beanName);
        return bean;
    }

    //初始化之后调用
    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("postProcessAfterInitialization..." + beanName);
        return bean;
    }
}
```

#### 5、总结

> bean的生命周期：bean创建 --> 初始化 --> 销毁
>  容器管理bean的生命周期，可以自定义初始化和销毁方法
>
>  构造（对象创建）
>       单实例：在容器启动的时候创建对象
>       多实例：在每次获取的时候创建对象
>
>  初始化：对象创建完成，并赋值好，调用初始化方法
>
>  销毁：单实例bean在容器关闭的时候调用销毁方法；多实例，容器不会管理这个bean，不调用销毁方法
>
>  1、指定初始化和销毁方法
>       指定init-method和destroy-method
>  2、通过让bean实现 InitializingBean(定义初始化逻辑)，DisposableBean(定义销毁逻辑)
>  3、使用JS250规范中的注解
>       @PostConstruct : 在bean创建完成并且属性赋值完成后执行初始化
>       @PreDestroy : 在容器销毁bean之前通知进行销毁工作
>  4、BeanPostProcessor: bean的后置处理器，在bean初始化前后进行一些处理工作
>       postProcessBeforeInitialization：在初始化之前工作
>       postProcessAfterInitialization：在初始化之后工作

## 三、属性赋值

```java
public class Person {
    //使用 @Value 赋值
    //1、基本数值
    //2、SpEL：#{}
    //3、${}取出配置文件中的值
    @Value("张三")
    private String name;
    @Value("#{20-2}")
    private Integer age;
}
```

加载配置文件中的值：

```java
/**
 * @PropertySource(value = "classpath:person.properties")
 * 读取外部配置文件保存到运行环境变量中，加载完配置文件中使用 ${} 取出配置文件中的值
 */
@Configuration
@PropertySource(value = "classpath:person.properties")
public class MyConfigOfPropertyValues {
}
```

```java
public class Person {
    //使用 @Value 赋值
    //1、基本数值
    //2、SpEL：#{}
    //3、${}取出配置文件中的值
    @Value("张三")
    private String name;
    @Value("#{20-2}")
    private Integer age;
    @Value("${person.nickName}")
    private String nickName;
}
```

## 四、自动装配

#### 1、@Autowired：自动注入

```java
/**
 * 自动装配
 *      Spring利用依赖注入，完成对IOC容器中各个组件的依赖关系赋值
 *
 * 1、@Autowired：自动注入
 *      1)、默认优先按照类型去容器找对应的组件applicationContext.getBean(BookService.class)
 *      2)、如果找到多个相同的组件，在将属性的名称作为组件的id去容器中查找
 *                  applicationContext.getBean("bookDao")
 *      3)、@Qualifier("bookDao")：使用@Qualifier指定需要装配的组件的id，而不是默认属性名
 *		4)、自动装配默认一定要将属性赋值好，没有就会报错
 *          可以使用@Autowired(required = false)，如果没有找到组件则不进行装配
 *      5)、@Primary：让Spring进行自动装配的时候，默认使用首选的bean，也可以使用@Qualifier("bookDao")
 *		装配指定的bean
 *     BookService {
 *          @Autowired
 *          private BookDao bookDao;
 *     }
 * 2、Spring还支持使用@Resource(JSR250)和@Inject(JSR330)[都是java规范的注解]
 *      @Resource :可以和 @Autowired 一样实现自动装配，默认按照组件的名称进行装配，不支持 @Primary 和 @Autowired(required = false) 功能
 *      @Inject :需要导入 javax.inject 的jar包，和 @Autowired 的功能一样
 *
 * 3、@Autowired：可以标注在 构造器、方法、属性 上。
 *
 * 4、自定义组件想要使用Spring容器底层的一些组件(ApplicationContext、BeanFactory)，
 *      可以让自定义组件实现 xxxAware 接口(ApplicationContextAware)
 *      在创建对象的时候会调用接口规定的方法注入相关组件
 *      xxxAware功能：使用xxxAwareProcessor
 */
@Configuration
@ComponentScan({"com.wf.service","com.wf.dao","com.wf.controller"})
public class MainConfigOfAutowired {
}
```

#### 2、@Profile

```java
/**
 * Profile：Spring提供的可以根据当前环境，动态激活和切换一系列组件的功能
 *
 * 开发环境、测试环境、生产环境
 *
 * @Profile : 指定组件在哪个环境的情况下才能被注册到容器中，不指定环境的情况下都能注册组件
 *
 * 加了环境标识的bean，只有相应的环境被激活的时候才能被注册到容器中，默认是default环境
 *      1)、加了环境标识的bean，只有相应的环境被激活的时候才能被注册到容器中，默认是default环境
 *      2)、将标识添加在配置类上，只有是指定的环境的时候，整个配置类中的所有配置才能生效
 *      3)、没有标识环境的bean在任何环境下都是加载的
 *
 * 切换环境：
 *      1)、使用命令行动态参数，在虚拟机参数位置修改(-Dspring.profiles.active=test)
 *      2)、代码实现
 */
@Configuration
@PropertySource("classpath:dbconfig.properties")
public class MainConfigOfProfile {
}
```

```java
@Test
public void test01(){
    //1、创建applicationContext
    AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext();
    //2、设置需要激活的环境
    applicationContext.getEnvironment().setActiveProfiles("test","dev");
    //3、注册主配置类
    applicationContext.register(MainConfigOfProfile.class);
    //4、启动刷新容器
    applicationContext.refresh();

    String[] beans = applicationContext.getBeanNamesForType(DataSource.class);
    for (String bean : beans) {
        System.out.println(bean);
    }
}
```

## 五、AOP原理

#### 1、AOP功能的实现

```java
/**
 * AOP：【动态代理】
 *  指在程序运行期间动态的将某段代码切入到指定方法的指定位置进行运行的编程方式
 *
 *  1、导入AOP模块:Spring AOP (spring-aspects)
 *  2、定义一个业务逻辑类
 *  3、定义一个日志切面类，切面类中的方法需要动态感知指定方法运行到哪里然后执行
 *      通知方法：
 *          前置通知(@Before)：在目标方法运行之前运行
 *          后置通知(@After)：在目标方法运行结束之后运行
 *          返回通知(@AfterReturning)：在目标方法正常返回之后运行
 *          异常通知(@AfterThrowing)：在目标方法运行出现异常之后运行
 *          环绕通知(@Around)：动态代理，手动推进目标方法运行
 *  4、给切面类的方法标注何时何地运行
 *  5、将切面类和业务逻辑类（目标方法所在的类）都加入到容器中
 *  6、告诉Spring哪个类是切面类（给切面类加 @Aspect 注解）
 *  7、给配置类中添加 @EnableAspectJAutoProxy 注解，启用 AspectJ 自动代理，开启基于注解的aop模式
 *
 *  在Spring中很多的 @EnableXXX 用来开启某一项功能来替代 xml 中的配置
 *
 *  AOP步骤：
 *      1)、将业务逻辑组件和切面类都加入到容器中，告诉Spring哪个是切面类（@Aspect）
 *      2)、在切面类上的每个通知方法上标注通知注解，告诉Spring何时何地运行（切入点表达式）
 *      3)、开启基于注解的AOP模式 @EnableAspectJAutoProxy（重要）
 */
@EnableAspectJAutoProxy
@Configuration
public class MainConfigOfAOP {
    @Bean
    public MathCalculator mathCalculator(){
        return new MathCalculator();
    }

    @Bean
    public LogAspects logAspects(){
        return new LogAspects();
    }
}
```

```java
@Aspect
public class LogAspects {
    //在目标方法之前切入；切入点表达式
    //JoinPoint一定要出现在参数列表的第一位
    @Before("pointCut()")
    public void logStart(JoinPoint joinPoint) {
        System.out.println(joinPoint.getSignature().getName() + "方法开始运行，方法参数为：" + Arrays.asList(joinPoint.getArgs()));
    }

    @After("pointCut()")
    public void logEnd(JoinPoint joinPoint) {
        System.out.println(joinPoint.getSignature().getName() + "方法执行结束");
    }
    
    @AfterReturning(value = "pointCut()", returning = "result")
    public void logReturn(JoinPoint joinPoint, Object result) {
        System.out.println(joinPoint.getSignature().getName() + "方法执行结束，返回结果为：" + result);
    }
    
    @AfterThrowing(value = "pointCut()", throwing = "exception")
    public void logException(JoinPoint joinPoint, Exception exception) {
        System.out.println(joinPoint.getSignature().getName() + "方法执行出错，错误信息" + exception);
    }
    
    //抽取公共的切入点表达式
    //1、本类引用 pointCut()
    //2、其它的切面引用 写方法的全名
    @Pointcut("execution(public int com.wf.aop.MathCalculator.*(..))")
    public void pointCut() {
    }
}
```

```java
public class MathCalculator {
    public int div(int i, int j) {
        return i / j;
    }
}
```

#### 2、AOP实现原理

```java
/* AOP原理：@EnableAspectJAutoProxy【看给容器中注册了什么组件，该组件什么时候工作，组件工作时的功能是什么】
 *  1、@EnableAspectJAutoProxy 是什么：
 *      使用 @Import(AspectJAutoProxyRegistrar.class) 给容器中导入 AspectJAutoProxyRegistrar
 *          利用 AspectJAutoProxyRegistrar 自定义给容器中注册 bean
 *          internalAutoProxyCreator = AnnotationAwareAspectJAutoProxyCreator
 *      给容器中注册一个 AnnotationAwareAspectJAutoProxyCreator：自动代理创建器
 *  2、AnnotationAwareAspectJAutoProxyCreator：
 *      AnnotationAwareAspectJAutoProxyCreator
 *          -> AspectJAwareAdvisorAutoProxyCreator
 *              -> AbstractAdvisorAutoProxyCreator
 *                  -> AbstractAutoProxyCreator implements SmartInstantiationAwareBeanPostProcessor, BeanFactoryAware
 *                  关注后置处理器（在bean初始化完成前后做的事情）、自动注入beanFactory
 *      通过分析继承树发现 AnnotationAwareAspectJAutoProxyCreator 具有 BeanPostProcessor 的特点
 *
 *      AbstractAutoProxyCreator.setBeanFactory()
 *      AbstractAutoProxyCreator.有后置处理器的逻辑
 *
 *      AbstractAdvisorAutoProxyCreator.setBeanFactory() -> initBeanFactory()
 *
 *      AnnotationAwareAspectJAutoProxyCreator.initBeanFactory
 */
```

AOP功能的实现需要依赖 `@EnableAspectJAutoProxy` 注解，只有添加该注解AOP功能才能生效。

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import(AspectJAutoProxyRegistrar.class)
public @interface EnableAspectJAutoProxy {}
```

在该注解中使用 `@Import(AspectJAutoProxyRegistrar.class)` 在容器中注册自定义的bean

```java
class AspectJAutoProxyRegistrar implements ImportBeanDefinitionRegistrar {
	/**
	 * Register, escalate, and configure the AspectJ auto proxy creator based on the value
	 * of the @{@link EnableAspectJAutoProxy#proxyTargetClass()} attribute on the importing
	 * {@code @Configuration} class.
	 */
	@Override
	public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
		//使用AopConfigUtils工具类给容器中注册AnnotationAutoProxyCreator
		AopConfigUtils.registerAspectJAnnotationAutoProxyCreatorIfNecessary(registry);

		AnnotationAttributes enableAspectJAutoProxy =
				AnnotationConfigUtils.attributesFor(importingClassMetadata, EnableAspectJAutoProxy.class);
		if (enableAspectJAutoProxy != null) {
			if (enableAspectJAutoProxy.getBoolean("proxyTargetClass")) {
				AopConfigUtils.forceAutoProxyCreatorToUseClassProxying(registry);
			}
			if (enableAspectJAutoProxy.getBoolean("exposeProxy")) {
				AopConfigUtils.forceAutoProxyCreatorToExposeProxy(registry);
			}
		}
	}
}
```

![](G:\云汇科技\笔记\浙思云汇\assets\AnnotationAwareAspectJAutoProxyCreator继承关系.png)

```java
/** 
 * AOP原理： @EnableAspectJAutoProxy【看给容器中注册了什么组件，该组件什么时候工作，组件工作时的功能是什么】
 *  1、@EnableAspectJAutoProxy 是什么：
 *      使用 @Import(AspectJAutoProxyRegistrar.class) 给容器中导入 AspectJAutoProxyRegistrar
 *          利用 AspectJAutoProxyRegistrar 自定义给容器中注册 bean
 *          internalAutoProxyCreator = AnnotationAwareAspectJAutoProxyCreator
 *      给容器中注册一个 AnnotationAwareAspectJAutoProxyCreator：自动代理创建器
 *  2、AnnotationAwareAspectJAutoProxyCreator：
 *      AnnotationAwareAspectJAutoProxyCreator
 *        -> AspectJAwareAdvisorAutoProxyCreator
 *          -> AbstractAdvisorAutoProxyCreator
 *             -> AbstractAutoProxyCreator implements SmartInstantiationAwareBeanPostProcessor, BeanFactoryAware
 *             关注后置处理器（在bean初始化完成前后做的事情）、自动注入beanFactory
 *      通过分析继承树发现 AnnotationAwareAspectJAutoProxyCreator 具有 BeanPostProcessor 的特点
 *
 *      AbstractAutoProxyCreator.setBeanFactory()
 *      AbstractAutoProxyCreator.有后置处理器的逻辑
 *
 *      AbstractAdvisorAutoProxyCreator.setBeanFactory() -> initBeanFactory()
 *
 *      AnnotationAwareAspectJAutoProxyCreator.initBeanFactory()
 */
```

**目标方法的执行流程：** 

```java
/** 
 * 方法调用流程：
 *  1、传入配置类，创建IOC容器
 *  2、注册配置类，调用refresh()方法刷新容器
 *  3、registerBeanPostProcessors(beanFactory);注册bean的后置处理器来方便拦截bean的创建
 *      1)、String[] postProcessorNames = beanFactory.getBeanNamesForType(BeanPostProcessor.class, true, false);
 *          先获取IOC容器中已经定义了的需要创建对象的所有BeanPostProcessor
 *      2)、给容器中加别的BeanPostProcessor
 *          beanFactory.addBeanPostProcessor(new BeanPostProcessorChecker(beanFactory, beanProcessorTargetCount));
 *      3)、优先注册实现了PriorityOrdered接口的BeanPostProcessor
 *      4)、其次再给容器中注册实现了Ordered接口的BeanPostProcessor
 *      5)、注册没实现优先级接口的BeanPostProcessor
 *      6)、注册BeanPostProcessor，实际上就是创建BeanPostProcessor对象，保存在容器中
 *          创建internalAutoProxyCreator的BeanPostProcessor【AnnotationAwareAspectJAutoProxyCreator】
 *          a、创建bean实例
 *          b、populateBean(beanName, mbd, instanceWrapper);给bean的属性赋值
 *          c、initializeBean(beanName, exposedObject, mbd);初始化bean
 *              1)、invokeAwareMethods(beanName, bean);处理Aware接口的方法回调
 *              2)、applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);应用后置处理器的BeforeInitialization()
 *              3)、invokeInitMethods(beanName, wrappedBean, mbd);执行自定义的初始化方法
 *              4)、applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);执行后置处理器的AfterInitialization
 *          d、BeanPostProcessor(AnnotationAwareAspectJAutoProxyCreator)创建成功， aspectJAdvisorFactory，aspectJAdvisorsBuilder
 *      7)、把BeanPostProcessor注册到BeanFactory中
 *          beanFactory.addBeanPostProcessor(postProcessor)
 * =============以上是创建 AnnotationAwareAspectJAutoProxyCreator 的过程=============
 *      AnnotationAwareAspectJAutoProxyCreator => InstantiationAwareBeanPostProcessor
 *  4、finishBeanFactoryInitialization(beanFactory);完成BeanFactory初始化工作，创建剩下的单实例bean
 *      1)、遍历获取容器中所有的Bean，依次创建对象，调用getBean(beanName)
 *          getBean(beanName) -> doGetBean(name, null, null, false); -> getSingleton(beanName);
 *      2)、创建bean
 *      【AnnotationAwareAspectJAutoProxyCreator 在所有bean创建之前会有一个拦截】
 *          a、先从缓存中获取当前bean，如果能获取到说明bean是之前被创建过的；直接使用，否则再创建，只要创建好的bean都会被缓存起来
 *          b、createBean(beanName, mbd, args);创建bean
 *              AnnotationAwareAspectJAutoProxyCreator 会在任何bean创建之前先尝试返回bean的实例
 *              【BeanPostProcessor是在Bean对象创建完成初始化前后进行调用的】
 *              【InstantiationAwareBeanPostProcessor在创建bean实例之前先尝试用后置处理器返回对象】
 *              1)、resolveBeforeInstantiation(beanName, mbdToUse);解析BeforeInstantiation
 *                  希望后置处理器在此能返回一个代理对象，如果能返回代理对象就直接使用，如果不能就继续
 *                  - 后置处理器先尝试返回对象
 *                      bean = applyBeanPostProcessorsBeforeInstantiation(targetType, beanName);
 *                      拿到所有后置处理器，如果是 InstantiationAwareBeanPostProcessor，就执行postProcessBeforeInstantiation方法
 *                      if (bean != null) {
 * 						    bean = applyBeanPostProcessorsAfterInitialization(bean, beanName);
 *                      }
 *              2)、doCreateBean(beanName, mbdToUse, args);真正的去创建一个bean实例
 *
 *  AnnotationAwareAspectJAutoProxyCreator => InstantiationAwareBeanPostProcessor 作用：
 *  1)、在每一个bean创建之前调用postProcessBeforeInstantiation()
 *      只关注 MathCalculator 和 logAspects 的创建
 *      a、判断当前bean是否在 advisedBeans 中(保存了所有需要增强的bean)
 *      b、判断当前bean是否是基础类型的 Advice、Pointcut、Advisor、AopInfrastructureBean 或者是否是切面(@Aspect) isAspect(beanClass)
 *      c、是否需要跳过【返回false】
 *          获取候选的增强器(切面里的通知方法)【List<Advisor> candidateAdvisors = findCandidateAdvisors();】
 *          每一个封装的通知方法的增强器是 InstantiationModelAwarePointcutAdvisor
 *          判断每一个增强器是否是 AspectJPointcutAdvisor 类型
 *  2)、创建对象 postProcessAfterInitialization
 *      return wrapIfNecessary(bean, beanName, cacheKey);//如果需要的情况下进行包装
 *      a、获取当前bean的所有增强器(通知方法)
 *          - 找到所有候选的增强器(找那些通知方法是需要切入当前bean方法的)
 *          - 获取能在当前bean使用的增强器
 *          - 给增强器排序
 *      b、保存当前bean在 advisedBeans 中
 *      c、如果当前bean需要增强，创建当前bean的代理对象
 *          - 获取所有增强器(通知方法)
 *          - 保存到 proxyFactory
 *          - 创建代理对象 Spring自动决定
 *              JdkDynamicAopProxy(config)
 *              ObjenesisCglibAopProxy(config)
 *      d、给容器返回当前组件增强的代理对象
 *      e、以后容器中获取到的就是这个组件的代理对象，执行目标方法的时候，代理对象就会执行通知方法
 *  3)、目标方法的执行
 *      容器中保存了组件的代理对象(cglib增强后的对象)，对象里保存了详细信息(如：增强器、目标对象...)
 *      a、CglibAopProxy.intercept();拦截目标方法的执行
 *      b、根据ProxyFactory对象获取拦截器链，获取目标方法的拦截器链
 *          List<Object> chain = this.advised.getInterceptorsAndDynamicInterceptionAdvice(method, targetClass);
 *          1)、List<Object> interceptorList保存所有拦截器
 *          2)、遍历所有的增强器，将其转为 Interceptor
 *              registry.getInterceptors(advisor)
 *          3)、将增强器转为 List<MethodInterceptor> interceptors
 *              如果是 MethodInterceptor 直接加入到集合中
 *              如果不是，使用 AdvisorAdapter 将增强器转为 MethodInterceptor
 *              转换完成返回 MethodInterceptor 数组
 *      c、如果没有拦截器链，直接执行目标方法
 *          拦截器链(每一个通知方法有被包装为方法拦截器，利用 MethodInterceptor 机制)
 *      d、如果有拦截器链，把需要执行的目标对象、目标方法、拦截器链等信息传入创建一个 CglibMethodInvocation 对象，并调用 proceed() 方法
 *          retVal = new CglibMethodInvocation(proxy, target, method, args, targetClass, chain, methodProxy).proceed();
 *      e、拦截器链的触发过程
 *          1)、如果没有拦截器直接执行目标方法，或者拦截器的索引和拦截器数组-1大小一样(指到了最后一个拦截器)
 *          currentInterceptorIndex 记录当前拦截器的索引
 *          2)、链式获取每一个拦截器，拦截器执行invoke()方法，每一个拦截器等待下一个拦截器执行完成返回以后再来执行，拦截器链的机制保证通知方法与目标方法的执行顺序
 *
 * 总结：
 *      1、@EnableAspectJAutoProxy 开启AOP功能
 *      2、@EnableAspectJAutoProxy 注解会给容器中注册一个 AnnotationAwareAspectJAutoProxyCreator 组件
 *      3、AnnotationAwareAspectJAutoProxyCreator 是一个后置处理器
 *      4、后置处理器的创建及工作流程
 *          1)、registerBeanPostProcessors(beanFactory);注册后置处理器，创建 AnnotationAwareAspectJAutoProxyCreator 对象
 *          2)、finishBeanFactoryInitialization(beanFactory);初始化剩下的单实例bean
 *              a、创建业务逻辑组件和切面组件
 *              b、AnnotationAwareAspectJAutoProxyCreator 拦截组件的创建过程
 *              c、组件创建完之后，postProcessAfterInitialization 判断组件是否需要增强
 *                  是：切面的通知方法包装成增强器(Advisor)，给业务逻辑组件创建一个代理对象
 *      5、执行目标方法
 *          1)、代理对象执行目标方法
 *          2)、CglibAopProxy.intercept();拦截目标方法的执行
 *              a、得到目标方法的拦截器链(增强器包装成 MethodInterceptor)
 *              b、利用拦截器的链式机制，依次进入每一个拦截器进行执行
 *              c、效果：
 *                  正常执行：前置通知 --> 目标方法 --> 后置通知 --> 返回通知
 *                  出现异常：前置通知 --> 目标方法 --> 后置通知 --> 异常通知
 */
```

#### 3、AOP原理总结

> 1、使用 @EnableAspectJAutoProxy 注解开启AOP功能
>
> 2、@EnableAspectJAutoProxy 注解会给容器中注册一个 AnnotationAwareAspectJAutoProxyCreator 组件
>
> 3、AnnotationAwareAspectJAutoProxyCreator 是一个后置处理器
>
> 4、后置处理器的创建和工作流程
>
> ​		1）、registerBeanPostProcessors(beanFactory) 注册后置处理器，创建 AnnotationAwareAspectJAutoProxyCreator 对象
>
> ​		2）、finishBeanFactoryInitialization(beanFactory);初始化剩下的单实例bean
>
> ​				a、创建业务逻辑组件和切面组件
>
> ​				b、AnnotationAwareAspectJAutoProxyCreator 拦截组件的创建过程
>
> ​				c、组件创建完之后，postProcessAfterInitialization 判断组件是否需要增强；是：切面的通知方法包装成增强器(Advisor)，给业务逻辑组件创建一个代理对象
>
> 5、执行目标方法
>
> ​		1)、代理对象执行目标方法
>
> ​		2)、CglibAopProxy.intercept();拦截目标方法的执行
>
> ​				a、得到目标方法的拦截器链(增强器包装成 MethodInterceptor)
>
> ​				b、利用拦截器的链式机制，依次进入每一个拦截器进行执行
>
> ​				c、效果：
>
> ​				正常执行：前置通知 --> 目标方法 --> 后置通知 --> 返回通知
>
> ​				出现异常：前置通知 --> 目标方法 --> 后置通知 --> 异常通知

## 六、声明式事务

```java
/**
 * 声明式事物
 * 环境搭建：
 * 1、导入相关依赖
 *      数据源、数据库驱动、Spring-jdbc模块
 * 2、配置数据源、JdbcTemplate操作数据库
 * 3、给方法上标注 @Transactional 注解，表示当前方法是一个事务方法
 * 4、在配置类上标注 @EnableTransactionManagement 开启事务功能
 * 5、配置事务管理器来控制事务
 * 原理：
 *  1）、利用 @Import(TransactionManagementConfigurationSelector.class) 给容器中导入组件
 *      导入两个组件 AutoProxyRegistrar、ProxyTransactionManagementConfiguration
 *  2）、AutoProxyRegistrar：
 *     给容器中注册一个 InfrastructureAdvisorAutoProxyCreator 组件
 *     利用后置处理器机制在对象创建以后，包装对象返回一个代理对象（增强器），代理对象执行方法利用拦截器链进行调用
 *
 *  3）、ProxyTransactionManagementConfiguration：
 *     1、事务增强器，事务增强器要用事务注解的信息 AnnotationTransactionAttributeSource 解析事务注解
 *     2、事务拦截器 TransactionInterceptor：保存了事务属性信息，事务管理器
 *         MethodInterceptor，在目标方法执行的时候，执行拦截器链
 *         事务拦截器：先获取事务相关的属性，再获取 PlatformTransactionManager
 *     3、执行目标方法，如果异常，获取到事务管理器，利用事务管理回滚操作
 *        如果正常，利用事务管理器提交事务
 */
@EnableTransactionManagement
@Configuration
public class TxConfig {
    @Bean
    public DataSource dataSource() throws PropertyVetoException {
        ComboPooledDataSource dataSource = new ComboPooledDataSource();
        dataSource.setUser("root");
        dataSource.setPassword("root");
        dataSource.setDriverClass("com.mysql.jdbc.Driver");
        dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/jdbc_db?useSSL=false");
        return dataSource;
    }
    @Bean
    public JdbcTemplate jdbcTemplate(DataSource dataSource) {
        JdbcTemplate jdbcTemplate = new JdbcTemplate();
        return jdbcTemplate;
    }
    //在容器中注册事务管理器
    @Bean
    public PlatformTransactionManager transactionManager(DataSource dataSource){
        return new DataSourceTransactionManager(dataSource);
    }
}
```

`@Transactional` 标注在方法上就可以对该方法进行事务控制。

## 七、扩展原理

#### 1、BeanFactoryPostProcessor

```java
/**
 * 扩展原理：
 * BeanPostProcessor：bean后置处理器，在bean创建对象初始化前后进行拦截工作的
 * BeanFactoryPostProcessor：beanFactory的后置处理器
 *  在beanFactory标准初始化之后调用，所有的 bean 定义已经保存加载到 beanFactory，但是bean的实例还未创建
 *
 *  1)、IOC容器创建对象
 *  2)、invokeBeanFactoryPostProcessors(beanFactory);执行beanFactoryPostProcessor
 *      a、直接在BeanFactory中找到所有类型是BeanFactoryPostProcessor的组件，并执行他们的方法
 *      b、在初始化创建其它组件前执行
 */
```

```java
@Component
public class MyBeanFactoryPostProcessor implements BeanFactoryPostProcessor {
    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {
        System.out.println("MyBeanFactoryPostProcessor...postProcessBeanFactory...");
        int count = beanFactory.getBeanDefinitionCount();
        System.out.println("当前beanFactory中有" + count + "个Bean");
        String[] names = beanFactory.getBeanDefinitionNames();
        System.out.println(Arrays.asList(names));
    }
}
```

#### 2、BeanDefinitionRegistryPostProcessor

```java
/** 
 * BeanDefinitionRegistryPostProcessor：
 *      postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry)
 *      在所有bean定义信息将要被加载，bean实例还未被创建的时候执行
 *      优先于BeanFactoryPostProcessor执行，利用BeanDefinitionRegistryPostProcessor可以给容器中再额外添加组件
 *  执行步骤：
 *  1)、创建IOC容器
 *  2)、refresh() --> invokeBeanFactoryPostProcessors(beanFactory);
 *  3)、从容器中获取到所有的BeanDefinitionRegistryPostProcessor组件
 *      a、依次触发所有的postProcessBeanDefinitionRegistry()
 *      b、再来触发postProcessBeanFactory()方法
 *  4)、再来从容器中找到BeanFactoryPostProcessor组件，然后依次触发postProcessBeanFactory()方法
 */
```

```java
@Component
public class MyBeanDefinitionRegistryPostProcessor implements BeanDefinitionRegistryPostProcessor {
    /**
     * BeanDefinitionRegistry：Bean定义信息的保存中心，beanFactory就是按照BeanDefinitionRegistry里面保存的每一个bean定义信息创建bean实例
     * @param registry Bean定义信息的保存中心
     * @throws BeansException
     */
    @Override
    public void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) throws BeansException {
        System.out.println("postProcessBeanDefinitionRegistry...bean的数量：" + registry.getBeanDefinitionCount());
        //BeanDefinition beanDefinition = new RootBeanDefinition(Dog.class);
        BeanDefinition beanDefinition = BeanDefinitionBuilder.rootBeanDefinition(Dog.class).getBeanDefinition();
        registry.registerBeanDefinition("hello", beanDefinition);
    }
    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {
        System.out.println("postProcessBeanFactory...bean的数量：" + beanFactory.getBeanDefinitionCount());
    }
}
```

#### 3、ApplicationListener

```java
/**
 * ApplicationListener：监听容器中发布的事件，事件驱动模型的开发
 *      public interface ApplicationListener<E extends ApplicationEvent> extends EventListener
 *      监听ApplicationEvent及其下面的子事件
 *  基于事件开发：
 *      1)、写一个监听器(ApplicationListener实现类)来监听某个事件(ApplicationEvent及其子类)
 *      2)、把监听器加入到容器中
 *      3)、只要容器中有相关事件的发布就能监听到事件
 *          ContextRefreshedEvent：容器刷新完成(所有的bean都完全创建)会发布这个事件
 *          ContextClosedEvent：关闭容器会发布该事件
 *      4)、发布一个事件  applicationContext.publishEvent()
 *
 *  原理：
 *      1、ContextRefreshedEvent：
 *          1)、容器创建对象，refresh();
 *          2)、容器刷新完成，finishRefresh();容器刷新完成会发布 ContextRefreshedEvent 事件
 *          3)、publishEvent(new ContextRefreshedEvent(this));发布一个事件
 *
 * 【事件发布流程】
 *  1、获取事件的多播器(派发器)，getApplicationEventMulticaster()
 *  2、multicastEvent()派发事件
 *  3、获取所有的ApplicationListener
 *      for (final ApplicationListener<?> listener : getApplicationListeners(event, type))
 *      如果有Executor，可以支持使用Executor进行异步派发  Executor executor = getTaskExecutor()
 *      同步方式直接执行listener方法：invokeListener(listener, event);
 *  4、拿到listener回调 onApplicationEvent(event) 方法
 *
 * 【事件多播器(事件派发器)】
 *  1、容器创建对象：refresh();
 *  2、initApplicationEventMulticaster();初始化ApplicationEventMulticaster
 *      1)、先从容器中找有没有 id="applicationEventMulticaster"的组件
 *      2)、如果没有 new SimpleApplicationEventMulticaster(beanFactory) 并且加入到容器中，
 *          在其他组件要派发事件的时候，自动注入 applicationEventMulticaster
 *
 * 【容器中的事件监听器】
 *  1、容器创建对象：refresh();
 *  2、注册监听器 registerListeners();
 *      从容器中拿到所有的监听器，并注册到 ApplicationEventMulticaster 中
 *      String[] listenerBeanNames = getBeanNamesForType(ApplicationListener.class,true,false);
 *      for (String listenerBeanName : listenerBeanNames) {
 * 			getApplicationEventMulticaster().addApplicationListenerBean(listenerBeanName);
 *      }
 *
 * 使用 @EventListener 注解让任意方法都能监听事件
 * 原理：
 *      使用 EventListenerMethodProcessor implements SmartInitializingSingleton 处理器来解析方法上的@EventListener注解
 *
 *  SmartInitializingSingleton 原理：
 *  1、IOC容器创建对象，并刷新容器 refresh();
 *  2、finishBeanFactoryInitialization(beanFactory);初始化剩下的单实例bean
 *      1)、先创建所有的单实例bean，getBean()
 *      2)、获取创建好的单实例bean判断是否是 SmartInitializingSingleton 类型，如果是就调用 smartSingleton.afterSingletonsInstantiated();
 *
 */
```

## 八、Servlet3.0

#### 1、@WebServlet

```java
@WebServlet("/hello")
public class HelloServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        resp.getWriter().write("Hello...");
    }
}
```

#### 2、Shared libraries(共享库) / runtimes pluggability(运行时插件)

Servlet 容器启动会扫描，当前应用里面每一个 jar 包的 ServletContainerInitializer 的实现。容器会在启动应用的时候，会扫描当前应用每一个jar包里面的

`META-INF/services/javax.servlet.ServletContainerInitializer` 指定的实现类，启动并运行这个实现类

#### 3、注册三大组件（Servlet、Filter、Listener）

使用 ServletContext 注册三大组件

```java
public class MyServletContainerInitializer implements ServletContainerInitializer {

    @Override
    public void onStartup(Set<Class<?>> set, ServletContext servletContext) throws ServletException {
        //注册Servlet   ServletRegistration.Dynamic
        ServletRegistration.Dynamic servlet = servletContext.addServlet("userServlet", new UserServlet());
        servlet.addMapping("/user");	//配置Servlet的映射信息
        //注册Filter   FilterRegistration.Dynamic
        FilterRegistration.Dynamic filter = servletContext.addFilter("userFilter", UserFilter.class);
        filter.addMappingForUrlPatterns(EnumSet.of(DispatcherType.REQUEST), true, "/*");
        //注册Listener
        servletContext.addListener();
    }
}
```

#### 4、SpringMVC整合

```txt
1、web容器在启动的时候，会扫描每个jar包下的 META-INF/services/javax.servlet.ServletContainerInitializer
2、加载这个文件指定的类 org.springframework.web.SpringServletContainerInitializer
3、spring 的应用一启动就会加载 WebApplicationInitializer 接口下的所有组件
4、为 WebApplicationInitializer 组件创建对象（组件不是接口，不是抽象类）
    1)、AbstractContextLoaderInitializer：创建根容器 createRootApplicationContext()
    2)、AbstractDispatcherServletInitializer：
        创建一个 web 的IOC容器：createServletApplicationContext()
        创建一个 DispatcherServlet：createDispatcherServlet(servletAppContext)
        将创建的 DispatcherServlet 添加到 ServletContext 中
    3)、AbstractAnnotationConfigDispatcherServletInitializer：注解方式配置的 DispatcherServlet 初始化器
        创建根容器：createRootApplicationContext()
            getRootConfigClasses();传入一个配置类
        创建web的IOC容器 createServletApplicationContext()
            获取配置类 getServletConfigClasses()

总结：
    以注解方式来启动 springMVC：继承 AbstractAnnotationConfigDispatcherServletInitializer 类
    实现抽象方法，指定 DispatcherServlet 的配置信息
```

```java
@ComponentScan(value = "com.wf", excludeFilters = {
        @ComponentScan.Filter(type = FilterType.ANNOTATION, classes = {Controller.class})
})
@Configuration
public class RootConfig {
}
```

```java
@ComponentScan(value = "com.wf", useDefaultFilters = false, includeFilters = {
        @ComponentScan.Filter(type = FilterType.ANNOTATION, classes = {Controller.class})
})
@Configuration
public class AppConfig {
}
```

```java
/**
 * web 容器启动的时候会创建对象，调用方法来初始化容器以及前端控制器
 */
public class MyWebAppInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {
    /**
     * 获取根容器的配置类（Spring的配置文件） 父容器
     * @return
     */
    @Override
    protected Class<?>[] getRootConfigClasses() {
        return new Class[]{RootConfig.class};
    }
    /**
     * 获取web容器的配置类（SpringMVC配置文件） 子容器
     * @return
     */
    @Override
    protected Class<?>[] getServletConfigClasses() {
        return new Class[]{AppConfig.class};
    }
    /**
     * 获取DispatcherServlet的映射信息
     * @return
     */
    @Override
    protected String[] getServletMappings() {
        //拦截所有请求，包括静态资源，但不包括*.jsp
        return new String[]{"/"};
    }
}
```

#### 5、定制 SpringMVC

```txt
定制SpringMVC
1)、@EnableWebMvc：开启SpringMVC定制配置功能，相当于配置文件中 <mvc:annotation-driven/>
2)、配置组件（视图解析器、试图映射、静态资源映射、拦截器...）
```

```java
@ComponentScan(value = "com.wf", useDefaultFilters = false, includeFilters = {
        @ComponentScan.Filter(type = FilterType.ANNOTATION, classes = {Controller.class})
})
@Configuration
@EnableWebMvc //开启SpringMVC定制配置功能
public class AppConfig implements WebMvcConfigurer {
    //定制视图解析器
    @Override
    public void configureViewResolvers(ViewResolverRegistry registry) {
        //默认所有的页面都从 /WEB-INF/ 下查找
        //registry.jsp();
        registry.jsp("/WEB-INF/views/", ".jsp");
    }
    //静态资源访问
    @Override
    public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
        configurer.enable();
    }
}
```

#### 6、Servlet3.0异步请求

> @WebServlet(value = "/async", asyncSupported = true) 需要Servlet支持异步请求

```java
@WebServlet(value = "/async", asyncSupported = true)
public class HelloAsyncServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        //1、支持异步处理 asyncSupported = true
        //2、开启异步模式
        System.out.println("主线程开始：" + Thread.currentThread());
        AsyncContext asyncContext = req.startAsync();
        //业务逻辑进行异步处理
        asyncContext.start(() -> {
            try {
                System.out.println("副线程开始：" + Thread.currentThread());
                sayHello();
                asyncContext.complete();
                //获取相应对象
                asyncContext.getResponse().getWriter().write("hello async");
                System.out.println("副线程结束：" + Thread.currentThread());
            } catch (IOException e) {
                e.printStackTrace();
            }
        });
        System.out.println("主线程结束：" + Thread.currentThread());
    }

    private void sayHello() {
        System.out.println(Thread.currentThread() + " processing...");
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

#### 7、SpringMVC异步处理

1. 方法返回 Callable 实现异步请求

    ```java
    @Controller
    public class AsyncController {
        /**
         * 1、控制器返回一个 Callable
         * 2、Spring异步处理，将 Callable 提交到 TaskExecutor 使用一个隔离的线程进行执行
         * 3、DispatcherServlet 和所有的Filter退出线程，但是response保持打开状态
         * 4、Callable 返回结果，SpringMVC将请求重新派发给容器，恢复之前的处理
         * 5、根据 Callable 返回的结果，SpringMVC继续进行处理流程（从收到请求到响应继续执行）
         *
         * 异步拦截器：
         *      1、原生API的AsyncListener
         *      2、SpringMVC：实现AsyncHandlerInterceptor接口写拦截器
         */
        @RequestMapping("/async01")
        @ResponseBody
        public Callable<String> async01(){
            System.out.println("主线程开始：" + Thread.currentThread());
            Callable<String> callable = () -> {
                System.out.println("副线程开始：" + Thread.currentThread());
                Thread.sleep(3000);
                System.out.println("副线程结束：" + Thread.currentThread());
                return "Callable<String> async01()";
            };
            System.out.println("主线程结束：" + Thread.currentThread());
            return callable;
        }
    }
    ```

2. 方法返回 DeferredResult 实现异步请求

    ```java
    @Controller
    public class AsyncController {
    
        @RequestMapping(value = "/createOrder")
        @ResponseBody
        public DeferredResult<Object> createOrder(){
            DeferredResult<Object> deferredResult = new DeferredResult<>((long)3000, "create fail");
            DeferredResultQueue.save(deferredResult);
            return deferredResult;
    
        }
    
        @RequestMapping(value = "/create")
        @ResponseBody
        public String create(){
            String orderId = UUID.randomUUID().toString();
            DeferredResult<Object> deferredResult = DeferredResultQueue.get();
            deferredResult.setResult(orderId);
            return "success ---> " + orderId;
        }
    }
    ```

    ```java
    public class DeferredResultQueue {
    
        private static Queue<DeferredResult<Object>> queue = new ConcurrentLinkedQueue<>();
    
        public static void save(DeferredResult<Object> deferredResult) {
            queue.add(deferredResult);
        }
    
        public static DeferredResult<Object> get() {
            return queue.poll();
        }
    }
    ```

    