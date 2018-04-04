---
layout:     post
title:     	Spring Boot 之 Cache Ehcache
subtitle:    "\"Spring Boot 之 Cache  Ehcache\""
date:       2017-09-26
author:     mrchang
header-img: img/post-bg-android.jpg
catalog: true
tags:
    - Spring Boot
    - Cache
    - Ehcache

---


## POM

	 <dependencies>
	        <dependency>
	            <groupId>org.springframework.boot</groupId>
	            <artifactId>spring-boot-starter-cache</artifactId>
	        </dependency>
	        <dependency>
	            <groupId>org.springframework.boot</groupId>
	            <artifactId>spring-boot-starter-actuator</artifactId>
	        </dependency>
	        <dependency>
	            <groupId>net.sf.ehcache</groupId>
	            <artifactId>ehcache</artifactId>
	        </dependency>
	        <dependency>
	            <groupId>com.alibaba</groupId>
	            <artifactId>fastjson</artifactId>
	            <version>1.2.12</version>
	        </dependency>
	   </dependencies>
	 
## SimpleApplication 

    @EnableCaching
    @EnableScheduling
    @SpringBootApplication
    public class SimpleApplication {
    
        public static void main(String[] args) {
            SpringApplication.run(SimpleApplication.class, args);
        }
    }
	
## application.properties

	spring.cache.type=ehcache
	spring.cache.ehcache.config=classpath:ehcache.xml
	
	

	
	
## ehcache.xml
		
	<ehcache name="spring_boot_auto_ehcache">
	    <diskStore path="java.io.tmpdir"/>
	
	    <cache name="CityService" maxElementsInMemory="10" eternal="true" overflowToDisk="false"/>
	</ehcache>
	
	
## CityInfo


	public class CityInfo implements Serializable {
	    private static final long serialVersionUID = 2845294956907027149L;
	
	    private int id;
	    private String city;
	    
## CityService

	@Component
	@CacheConfig(cacheNames = "CityService")
	public class CityService {
	    Logger logger = LogManager.getLogger(getClass());
	
	    @Cacheable
	    public CityInfo getCity(int id, String city) {
	        logger.info("id: {}, city: {}", id, city);
	        return new CityInfo(id, city);
	    }
	}
	
	
## TaskJob

	@Component
	public class TaskJob {
	    private static final List<String> list = Arrays.asList("北京市", "上海市", "天津市", "重庆市", "河北省", "山西省", "内蒙古自治区", "辽宁省",
	            "吉林省", "黑龙江", "江苏省", "浙江省", "安徽省", "福建省", "江西省", "山东省", "河南省", "湖北省", "湖南省", "广东省", "广西自治区", "海南省", "四川省",
	            "贵州省", "云南省", "西藏自治区", "陕西省", "甘肃省", "青海省", "宁夏自治区", "新疆自治区", "香港特别行政区", "澳门特别行政区", "台湾省");
	    Logger logger = LogManager.getLogger(getClass());
	    @Autowired
	    private CityService cityService;
	
	    /**
	     * Job
	     */
	    @Scheduled(fixedDelay = 500)
	    public void retrieveCountry() {
	        int index = new Random().nextInt(list.size());
	        String city = find(index);
	
	        //两条log  有缓存记录的时候 输出一条 没有缓存记录的时候输出2条
	        CityInfo info = cityService.getCity(index, city);
	
	        logger.info("city: {}", JSON.toJSONString(info));
	    }
	
	
	    
	    
## 参考

**代码：https://github.com/changdaye/spring-boot-demo/tree/master/spring-boot-cache-ehcache**