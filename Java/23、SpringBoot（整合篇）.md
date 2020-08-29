## 一、SpringBoot 与缓存

#### 1、JSR 107：

- **CachingProvider：** 定义了创建、配置、获取、管理和控制多个 **CacheManager** 。一个应用可以在运行期间访问多个 CachingProvider。
- **CacheManager：** 定义了创建、配置、获取、管理和控制多个唯一命名的 Cache，这些 Cache 存在于 CacheManager 的上下文中，一个 CacheManager 仅被一个 CachingProvider 所拥有。
- **Cache：** 是一个类似Map的数据结构并临时存储以 Key 为索引的值。一个 Cache 仅被一个 CacheManager 所拥有。
- **Entry：** 是一个存储在 Cache 中的 Key-Value 对。
- **Expiry：** 每一个存储在 Cache 中的条目有一个定义的有效期，一旦超过这个有效期，条目为过期状态，一旦过期条目不可访问、更新和删除。缓存有效期可以通过 ExpiryPokicy 设置。

#### 2、Spring 缓存抽象：

|                | 解释                                                       |
| :------------- | ---------------------------------------------------------- |
| Cache          | 缓存接口，定义缓存操作。实现有：RedisCache、EhCacheCache等 |
| CacheManager   | 缓存管理器，管理各种缓存（Cache）组件                      |
| @Cacheable     | 主要针对方法配置，能够根据方法的请求参数对其结果进行缓存   |
| @CacheEvict    | 清空缓存                                                   |
| @CachePut      | 保证方法被调用，有更新缓存数据                             |
| @EnableCaching | 开启基于注解的缓存                                         |
| keyGenerator   | 缓存数据时 key 生成策略                                    |
| serialize      | 缓存数据时value序列化策略                                  |

缓存实现步骤：

1）、开启基于注解的缓存： @EnableCaching

```java
@SpringBootApplication
@EnableCaching
public class SpringBoot01CacheApplication {
    public static void main(String[] args) {
        SpringApplication.run(SpringBoot01CacheApplication.class, args);
    }
}
```

2）、标注缓存注解：

```java
/**
 * @Cacheable 将方法的缓存结果进行缓存，再次查询相同的数据直接从缓存中获取，不用连接数据库
 *
 * cacheManager：管理多个 Cache 组件，对缓存的真正操作在 Cache 组件中，每一个缓存组件有自己唯一一个名字
 * 属性：
 *      cacheNames/value：指定缓存的名字；将方法的返回值放在哪个缓存中，是数组的方式，可以指定多个缓存
 *      key：缓存数据使用的key。默认是使用方法参数的值  方法参数值 - 方法返回值。也可以使用 SpEL 表达式
 *          #id 参数id的值  #a0 #p0 #root.args[0]
 *      keyGenerator：key的生成器，默认是使用参数的值，也可以自己指定key的生成器的组件id
 *          key和keyGenerator二选一使用
 *      cacheManager：指定缓存管理器，或者指定缓存解析器 cacheResolver 二选一
 *      condition：指定符合条件的情况下才缓存 condition = "#id > 0"
 *      unless：否定缓存，当unless指定的条件为true方法的返回值就不会被缓存，可以获取到结果进行判断 #result
 *          unless = "#result == null"
 *      sync：是否使用异步模式
 */
@Cacheable(cacheNames = "emp", condition = "#id > 0", unless = "#result == null")
public Employee getEmpById(Integer id){
    System.out.println("查询 " + id + " 号员工");
    return employeeMapper.getEmpById(id);
}
```

实现缓存原理：

```java
原理：
	1、自动配置类： CacheAutoConfiguration
	2、缓存的配置类：
  		org.springframework.boot.autoconfigure.cache.GenericCacheConfiguration
  		org.springframework.boot.autoconfigure.cache.JCacheCacheConfiguration
  		org.springframework.boot.autoconfigure.cache.EhCacheCacheConfiguration
  		org.springframework.boot.autoconfigure.cache.HazelcastCacheConfiguration
   		org.springframework.boot.autoconfigure.cache.InfinispanCacheConfiguration
   		org.springframework.boot.autoconfigure.cache.CouchbaseCacheConfiguration
   		org.springframework.boot.autoconfigure.cache.RedisCacheConfiguration
   		org.springframework.boot.autoconfigure.cache.CaffeineCacheConfiguration
   		org.springframework.boot.autoconfigure.cache.SimpleCacheConfiguration【默认】
   		org.springframework.boot.autoconfigure.cache.NoOpCacheConfiguration
	3、默认生效的配置类
   		SimpleCacheConfiguration
	4、给容器中注册了一个 ConcurrentMapCacheManager
	5、可以获取和创建 ConcurrentMapCache 类型的缓存组件；其作用是将数据保存在ConcurrentMap

运行流程： @Cacheable
	1、方法运行之前，先去查询 Cache（缓存组件），按照 cacheNames 指定的名字获取
	（CacheManager先获取相应的缓存），第一次获取缓存如果没有Cache组件会自动创建
	2、去 Cache 中查找缓存的内容，使用key去查找，默认就是方法的参数。key是按照某种策略生成的，默认是使用keyGenerator生成的，默认使用SimpleKeyGenerator生成key
	SimpleKeyGenerator生成key的策略：
		如果没有参数：key=new SimpleKey()
   		如果有一个参数：key=参数值
   		如果有多个参数：key=new SimpleKey(params)
	3、没有查到缓存就调用目标方法
	4、将目标方法返回的结果放进缓存里
	@Cacheable 标注的方法执行之前先来检查缓存中有没有这个数据，默认按照参数的值作为 key 去查询缓存，如果没有就运行方法并将结果放入缓存，以后再来调用就可以直接使用缓存中的数据

 核心：
    1）、使用CacheManager【ConcurrentMapCacheManager】按照名字得到Cache【ConcurrentMapCache】组件
    2）、key使用keyGenerator生成的，默认是SimpleKeyGenerator
```

3）、缓存key可以使用的SpEL表达式：

| 名字          | 位置               | 描述                                                         | 示例                 |
| ------------- | ------------------ | ------------------------------------------------------------ | -------------------- |
| methodName    | root object        | 当前被调用的方法名                                           | #root.methodName     |
| method        | root object        | 当前被调用的方法                                             | #root.method.name    |
| target        | root object        | 当前被调用的目标对象                                         | #root.target         |
| targetClass   | root object        | 当前被调用的目标对象类                                       | #root.targetClass    |
| args          | root object        | 当前被调用的方法的参数列表                                   | #root.args[0]        |
| caches        | root object        | 当前方法调用使用的缓存列表（如@Cacheable(value={"cache1",   "cache2"})），则有两个cache | #root.caches[0].name |
| argument name | evaluation context | 方法参数的名字. 可以直接 #参数名 ，也可以使用 #p0或#a0 的形式，0代表参数的索引； | #iban 、 #a0 、  #p0 |
| result        | evaluation context | 方法执行后的返回值（仅当方法执行之后的判断有效，如‘unless’，’cache put’的表达式 ’cache evict’的表达式beforeInvocation=false） | #result              |

**自定义key生成策略：** 

```java
@Configuration
public class MyCacheConfig {

    @Bean("myKeyGenerator")
    public KeyGenerator keyGenerator(){
        return (target, method, params) -> method.getName() + "[" + Arrays.asList(params) + "]";
    }
}
//在需要缓存的位置使用 keyGenerator="myKeyGenerator" 即可使用自己自定义的key生成策略
```

**@CachePut 缓存更新：** 

```java
/**
 * 缓存更新
 * @CachePut ：既调用方法，有更新缓存数据；修改了数据库的某条数据后更新缓存
 *
 * 运行时机：
 *      1、调用目标方法
 *      2、将目标方法缓存起来
 * 注意：使用缓存更新需要指定要更新的 key 值，要更新的 key 和 已经缓存的 key 一致才能达到更新缓存的目的
 */
@CachePut(value = "emp", key = "#result.id")
public Employee updateEmp(Employee employee) {
    employeeMapper.updateEmp(employee);
    return employee;
}
```

**@CacheEvict 缓存清除：** 

```java
/**
 * 缓存清除：
 *      key：指定要清除的缓存的数据
 * allEntries：是否要清除所有的缓存，默认为 false
 * beforeInvocation：缓存的清除是否在方法执行之前执行，默认是false，代表是在方法执行之后执行的
 * 如果方法运行出现错误则不会清除缓存
 */
@CacheEvict(value = "emp", key = "#id")
public void deleteEmp(Integer id){
    employeeMapper.deleteEmp(id);
}
```

**定义复杂缓存规则：** 

```java
/**
 * @Caching 可以使用复杂的缓存规则
 */
@Caching(
        cacheable = {
                @Cacheable(value = "emp", key = "#lastName")
        },
        put = {
                @CachePut(value = "emp", key = "#result.id"),
                @CachePut(value = "emp", key = "#result.email")
        }
)
public Employee getEmpByLastName(String lastName){
    return employeeMapper.getEmpByLastName(lastName);
}
```

**@CacheConfig：** 抽取缓存的公共配置

#### 3、使用 Redis 缓存：

##### 1）、引入redis相关依赖：

```xml
<!--使用Redis做缓存-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

##### 2）、配置redis host：

```yaml
spring: 
	redis:
    	host: 192.168.174.131
```

##### 3）、使用redis缓存：

```java
public class SpringBoot01CacheApplicationTests {
    @Autowired
    StringRedisTemplate stringRedisTemplate;//操作k-v都是字符串的
    @Autowired
    RedisTemplate redisTemplate;//k-v都是对象的
    @Autowired
    EmployeeMapper employeeMapper;
    /**
     * Redis常见的五大数据类型
     * String（字符串）、List（列表）、Set（集合）、Hash（散列）、ZSet（有序集合）
     */
    @Test
    public void contextLoads() {
        //给Redis中保存数据
        //stringRedisTemplate.opsForValue().append("msg", "hello");
        //System.out.println(stringRedisTemplate.opsForValue().get("msg"));
        stringRedisTemplate.opsForList().leftPush("myList", "1");
        stringRedisTemplate.opsForList().leftPush("myList", "2");
        stringRedisTemplate.opsForList().leftPush("myList", "3");
    }
    /**
     * 测试保存对象
     */
    @Test
    public void test01(){
        Employee employee = employeeMapper.getEmpById(1);
        //如果保存对象，使用jdk序列化机制，序列化后的数据保存到redis中
        redisTemplate.opsForValue().set("emp-02", employee);
        //将数据以json的方式保存
        //1、自己将对象转换成json
        //2、redisTemplate有默认的序列化规则
    }
}
```

##### 4）、自定义 RedisCacheManager 的值序列化方式为json序列化：

```java
@Bean
public RedisCacheManager redisCacheManager(RedisConnectionFactory redisConnectionFactory){
    //初始化一个RedisCacheWriter
    RedisCacheWriter redisCacheWriter = RedisCacheWriter.nonLockingRedisCacheWriter(redisConnectionFactory);
    //设置CacheManager的值序列化方式为json序列化
    RedisSerializer<Object> jsonSerializer = new GenericJackson2JsonRedisSerializer();
    RedisSerializationContext.SerializationPair<Object> pair = RedisSerializationContext.SerializationPair
            .fromSerializer(jsonSerializer);
    RedisCacheConfiguration defaultCacheConfig=RedisCacheConfiguration.defaultCacheConfig()
            .serializeValuesWith(pair);
    //初始化RedisCacheManager
    return new RedisCacheManager(redisCacheWriter, defaultCacheConfig);
}
```

## 二、SpringBoot 与消息

消息服务中的两大概念： 消息代理（message broker）和目的地（destination），当消息发送者发送消息后，将由消息代理接管，消息代理保证消息传递到指定目的地。

**消息队列主要有两种形式的目的地：** 

- 队列（queue）：点对点消息通信（point-to-point）
- 主题（topic）：发布（publish）/订阅（subscribe）消息通信

**点对点式：** 

- 消息发送者发送消息，消息代理将其放入一个队列中，消息接收者从队列中获取消息内容，消息读取后被移除队列
- 消息只有唯一个发送者和接受者，但并不是说只能有一个接收者（一个消息可以有 A、B、C、D 等多个接收者，但最终只能有一个会接受这条消息，被称为消息接受者）。

**发布订阅式：** 

- 发送者（发布者）发送消息到主题，多个接收者（订阅者）监听（订阅）这个主题，那么就会在消息到达时同时收到消息。

**JMS（Java Message Service）Java 消息服务：** 

- 基于 JVM 消息代理的规范。ActiveMQ、HornetMQ是JMS实现

**AMQP（Advanced Message Queuing Protocol）：** 

- 高级消息队列协议，也是一个消息代理的规范，兼容JMS
- RabbitMQ是AMQP的实现