---
layout:     post
title:     	Spring Boot 之 Cache Redis
subtitle:    "\"Spring Boot 之 Cache   Redis\""
date:       2017-09-26
author:     mrchang
header-img: img/post-bg-coffee.jpeg
catalog: true
tags:
    - Spring Boot
    - Cache
    - Redis

---


## POM

	<dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-redis</artifactId>
        </dependency>
        <dependency>
            <groupId>redis.clients</groupId>
            <artifactId>jedis</artifactId>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.12</version>
        </dependency>
    </dependencies>	 
	 
	
## application.properties
	
	# REDIS (RedisProperties)
	spring.redis.database=4
	spring.redis.host=103.37.124.108
	#spring.redis.password=123456
	spring.redis.pool.max-active=8
	spring.redis.pool.max-idle=8
	spring.redis.pool.max-wait=-1
	spring.redis.pool.min-idle=0
	spring.redis.port=6379
	spring.redis.timeout=0
	
	

	
	
## SimpleApplication
		

	@EnableCaching
	@EnableScheduling
	@SpringBootApplication
	public class SimpleApplication {
	
	    public static void main(String[] args) {
	        SpringApplication.run(SimpleApplication.class, args);
	    }
	}
	
	
	    
## RedisConfig

	@Configuration
	@EnableCaching
	public class RedisConfig extends CachingConfigurerSupport {
	
	    /**
	     * 设置统一的生成key的方式
	     * @return
	     */
	    @Bean
	    public KeyGenerator wiselyKeyGenerator() {
	        return new KeyGenerator() {
	            @Override
	            public Object generate(Object target, Method method, Object... params) {
	                StringBuilder sb = new StringBuilder();
	                sb.append(target.getClass().getName());
	                sb.append("-");
	                sb.append(method.getName());
	                for (Object obj : params) {
	                    sb.append(obj.toString());
	                }
	                return sb.toString();
	            }
	        };
	
	    }
	
	    /**
	     * 设置统一失效时间
	     * @param redisTemplate
	     * @return
	     */
	    @Bean
	    public CacheManager cacheManager(
	            @SuppressWarnings("rawtypes") RedisTemplate redisTemplate) {
	        RedisCacheManager redisCacheManager = new RedisCacheManager(redisTemplate);//设置统一实效时间
	        redisCacheManager.setDefaultExpiration(604800);
	        redisCacheManager.setTransactionAware(true);
	        return redisCacheManager;
	    }
	
	    /**
	     * 设置统一序列化
	     * @param factory
	     * @return
	     */
	    @Bean
	    public RedisTemplate<String, String> redisTemplate(
	            RedisConnectionFactory factory) {
	        StringRedisTemplate template = new StringRedisTemplate(factory);
	        Jackson2JsonRedisSerializer jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);
	        ObjectMapper om = new ObjectMapper();
	        om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
	        om.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
	        jackson2JsonRedisSerializer.setObjectMapper(om);
	        template.setValueSerializer(jackson2JsonRedisSerializer);
	        template.afterPropertiesSet();
	        return template;
	    }
	}

	
## CityService


	@Component
	@CacheConfig(cacheNames = "CityService")
	public class CityService {
	    Logger logger = LogManager.getLogger(getClass());
	
	    @Cacheable(keyGenerator = "wiselyKeyGenerator")//使用自定义key
	    public CityInfo getCity(int id, String city) {
	        logger.info("id: {}, city: {}", id, city);
	        return new CityInfo(id, city);
	    }
	
	    @Cacheable(key = "#id + '_'  + #city")//放入缓存
	    public CityInfo getCity1(int id, String city) {
	        logger.info("id: {}, city: {}", id, city);
	        return new CityInfo(id, city);
	    }
	
	
	    @CachePut(key = "#id + '_'  + #city")//刷新缓存
	    public void addCity(int id, String city) {
	        logger.info("id: {}, city: {}", id, city);
	        new CityInfo(id, city);
	    }
	
	
	}	
	

## Cache注解详解

回过头来我们再来看，这里使用到的两个注解分别作了什么事情。

1. `@CacheConfig`：主要用于配置该类中会用到的一些共用的缓存配置。
	
	* 在这里`@CacheConfig(cacheNames = "users")`：配置了该数据访问对象中返回的内容将存储于名为`users`的缓存对象中，我们也可以不使用该注解，直接通过`@Cacheable`自己配置缓存集的名字来定义

2.  @Cacheable：配置了findByName函数的返回值将被加入缓存。同时在查询时，会先从缓存中获取，若不存在才再发起对数据库的访问。该注解主要有下面几个参数：

	* `value`、`cacheNames`：两个等同的参数（`cacheNames`为Spring 4新增，作为`value`的别名），用于指定缓存存储的集合名。由于Spring 4中新增了`@CacheConfig`，因此在Spring 3中原本必须有的`value`属性，也成为非必需项了
	* `key`：缓存对象存储在Map集合中的key值，非必需，缺省按照函数的所有参数组合作为key值，若自己配置需使用SpEL表达式，比如：`@Cacheable(key = "#p0")`：使用函数第一个参数作为缓存的key值，[更多关于SpEL表达式的详细内容可参考官方文档
](https://docs.spring.io/spring/docs/current/spring-framework-reference/html/cache.html#cache-spel-context)

	* `condition`：缓存对象的条件，非必需，也需使用SpEL表达式，只有满足表达式条件的内容才会被缓存，比如：`@Cacheable(key = "#p0", condition = "#p0.length() < 3")`，表示只有当第一个参数的长度小于3的时候才会被缓存，若做此配置上面的AAA用户就不会被缓存，读者可自行实验尝试。
	* `unless`：另外一个缓存条件参数，非必需，需使用SpEL表达式。它不同于`condition`参数的地方在于它的判断时机，该条件是在函数被调用之后才做判断的，所以它可以通过对result进行判断。
	* `keyGenerator`：用于指定key生成器，非必需。若需要指定一个自定义的key生成器，我们需要去实现`org.springframework.cache.interceptor.KeyGenerator`接口，并使用该参数来指定。需要注意的是：**该参数与`key`是互斥的
**
	* `cacheManager`：用于指定使用哪个缓存管理器，非必需。只有当有多个时才需要使用
	* `cacheResolver`：用于指定使用那个缓存解析器，非必需。需通过`org.springframework.cache.interceptor.CacheResolver`接口来实现自己的缓存解析器，并用该参数指定。

除了这里用到的两个注解之外，还有下面几个核心注解：

3. `@CachePut`：配置于函数上，能够根据参数定义条件来进行缓存，它与`@Cacheable`不同的是，它每次都会真是调用函数，所以主要用于数据新增和修改操作上。它的参数与`@Cacheable`类似，具体功能可参考上面对`@Cacheable`参数的解析
4. `@CacheEvict`：配置于函数上，通常用在删除方法上，用来从缓存中移除相应数据。除了同`@Cacheable`一样的参数之外，它还有下面两个参数：
	* `allEntries`：非必需，默认为false。当为true时，会移除所有数据
	* `beforeInvocation`：非必需，默认为false，会在调用方法之后移除数据。当为true时，会在调用方法之前移除数据。
	
	    
	    
## 参考

**代码：https://github.com/changdaye/spring-boot-demo/tree/master/spring-boot-cache-redis**