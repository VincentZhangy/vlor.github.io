---
layout:     post
title:     	Spring Boot 之 Mybatis
subtitle:    "\"Spring Boot 之 Mybatis\""
date:       2017-09-25
author:     mrchang
header-img: img/post-bg-re-vs-ng2.jpg
catalog: true
tags:
    - Spring Boot
    - Mybatis

---


## POM

	 <dependencies>
	        <!-- Mybatis -->
	        <dependency>
	            <groupId>org.mybatis.spring.boot</groupId>
	            <artifactId>mybatis-spring-boot-starter</artifactId>
	            <version>1.1.1</version>
	        </dependency>
	       
	        <!-- MYSQL -->
	        <dependency>
	            <groupId>mysql</groupId>
	            <artifactId>mysql-connector-java</artifactId>
	        </dependency>
	        <dependency>
	            <groupId>com.alibaba</groupId>
	            <artifactId>druid</artifactId>
	            <version>1.0.20</version>
	        </dependency>
	        <dependency>
	            <groupId>com.alibaba</groupId>
	            <artifactId>fastjson</artifactId>
	            <version>1.2.12</version>
	        </dependency>
	 </dependencies>
	 
	 
	
## application.properties

	#MYBATIS
	mybatis.type-aliases-package=mybatis.domain
	mybatis.mapper-locations=classpath*:/mapper/*Mapper.xml
	mybatis.configuration.map-underscore-to-camel-case=true
	mybatis.configuration.use-generated-keys=true
	mybatis.configuration.default-fetch-size=100
	mybatis.configuration.default-statement-timeout=30
	
	#DATASOURCE
	spring.datasource.schema=classpath:init-sql/schema.sql
	spring.datasource.continueOnError=true
	spring.datasource.type=com.alibaba.druid.pool.DruidDataSource
	spring.datasource.url=jdbc:mysql://localhost/demo-schema
	spring.datasource.username=root
	spring.datasource.password=123456
	spring.datasource.driver-class-name=com.mysql.jdbc.Driver
	spring.datasource.initialSize=5
	spring.datasource.minIdle=5
	spring.datasource.maxActive=20
	spring.datasource.maxWait=60000
	spring.datasource.timeBetweenEvictionRunsMillis=60000
	spring.datasource.validationQuery=SELECT 1
	spring.datasource.testWhileIdle=true
	spring.datasource.testOnBorrow=false
	spring.datasource.testOnReturn=false
	spring.datasource.poolPreparedStatements=true
	spring.datasource.maxPoolPreparedStatementPerConnectionSize=20
	spring.datasource.filters=stat
	spring.datasource.connectionProperties=druid.stat.mergeSql=true;druid.stat.slowSqlMillis=5000
	
	
## schema.sql

	SET FOREIGN_KEY_CHECKS=0;
	
	-- ----------------------------
	-- Table structure for `boot_user`
	-- ----------------------------
	DROP TABLE IF EXISTS `boot_user`;
	CREATE TABLE `boot_user` (
	  `id` int(11) NOT NULL AUTO_INCREMENT,
	  `name` varchar(32) DEFAULT NULL,
	  `tel` varchar(11) DEFAULT NULL,
	  `create_time` datetime DEFAULT NULL,
	  PRIMARY KEY (`id`)
	) ENGINE=InnoDB AUTO_INCREMENT=3 DEFAULT CHARSET=utf8;
	
	-- ----------------------------
	-- Records of boot_user
	-- ----------------------------
	INSERT INTO `boot_user` VALUES ('1', 'klay', '13799008800', '2016-06-27 00:01:39');
	INSERT INTO `boot_user` VALUES ('2', 'Tome', '18988991234', '2016-06-27 00:35:28');
	
	
## Controller
		
	@Controller
	public class IndexController {
	    @Autowired
	    private UserService userService;
	
	    /**
	     * 查询用户
	     *
	     * @return
	     */
	    @ResponseBody
	    @RequestMapping("finduser")
	    public String findUser() {
	        return JSON.toJSONString(userService.findAll());
	    }
	
	    /**
	     * findById
	     *
	     * @param id
	     * @return
	     */
	    @ResponseBody
	    @RequestMapping("user/{id}")
	    public String findById(@PathVariable int id) {
	        return JSON.toJSONString(userService.findOne(id));
	    }
	}
	
	
## Service

	@Service
    @Transactional
    public class UserServiceImpl implements UserService {
        @Autowired
        private UserMapper userMapper;
    
        @Override
        public List<UserInfo> findAll() {
            return userMapper.findAll();
        }
    
        @Override
        public UserInfo findOne(int id) {
            return userMapper.findOne(id);
        }
    
    }
	
	
## Mapper||Dao


	@Mapper
	public interface UserMapper {
	
	    /**
	     * findOne
	     *
	     * @param id
	     * @return
	     */
	    @Select(value = "select *from boot_user where id=#{id}")
	    UserInfo findOne(int id);
	
	    /**
	     * findAll
	     *
	     * @return
	     */
	    List<UserInfo> findAll();
	}
	
	
## Mapper.xml

	<?xml version="1.0" encoding="UTF-8"?>
	<!DOCTYPE mapper
	        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
	        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
	<mapper namespace="mybatis.mapper.UserMapper">
	
	    <!-- findAll -->
	    <select id="findAll" resultType="UserInfo">
	        select *from boot_user
	    </select>
	</mapper>
	
## domain||pojo
	
	public class UserInfo implements Serializable {
	    private static final long serialVersionUID = 6519997700281088880L;
	
	    private int id;
	
	    private String name;
	
	    private String tel;
	
	    @JSONField(format = "yyyy-MM-dd")
	    private Date createTime;
	    
	    ...getter setter
	    
	    
## 参考

**代码：https://github.com/changdaye/spring-boot-demo/tree/master/spring-boot-mybatis**