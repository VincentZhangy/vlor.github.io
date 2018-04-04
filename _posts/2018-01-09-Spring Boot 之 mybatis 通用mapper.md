---
layout:     post
title:     	Spring Boot 之 mybatis 通用mapper 
subtitle:    "\"Spring Boot 之 mybatis 通用mapper \""
date:       2018-01-09
author:     Mr Chang
header-img: img/post-bg-e2e-ux.jpg
catalog: true
tags:
    - Spring Boot
    - 通用mapper

---


# 项目依赖

	<!--mybatis-->
	<dependency>
	    <groupId>org.mybatis.spring.boot</groupId>
	    <artifactId>mybatis-spring-boot-starter</artifactId>
	    <version>1.3.1</version>
	</dependency>
	<!--mapper-->
	<dependency>
	    <groupId>tk.mybatis</groupId>
	    <artifactId>mapper-spring-boot-starter</artifactId>
	    <version>1.1.4</version>
	</dependency>
	<!--pagehelper-->
	<dependency>
	    <groupId>com.github.pagehelper</groupId>
	    <artifactId>pagehelper-spring-boot-starter</artifactId>
	    <version>1.2.1</version>
	</dependency>
	
	
# application.properties 配置


	spring.profiles.active=dev
	
	# mybatis 配置
	mybatis.type-aliases-package=cn.org.jetbrains.universalmapper.pojo
	mybatis.mapper-locations=classpath:mapper/*.xml
	# 通用 Mapper 配置
	mapper.mappers=cn.org.jetbrains.universalmapper.util.MyMapper
	mapper.not-empty=false
	mapper.identity=MYSQL
	# 分页插件配置
	pagehelper.helperDialect=mysql
	pagehelper.reasonable=true
	pagehelper.supportMethodsArguments=true
	pagehelper.params=count=countSql
	
	mybatis.config-location=classpath:config/mybatis-config.xml
	
	server.port=9090
	debug=true
	logging.level.root=debug
	logging.level.tk.mybatis.springboot.mapper=trace
	spring.datasource.url=jdbc:mysql://localhost:3306/test
	spring.datasource.username=root
	spring.datasource.password=changdaye
	spring.datasource.driver-class-name=com.mysql.jdbc.Driver
	
# MyMapper

	public interface MyMapper <T> extends Mapper<T>, MySqlMapper<T> {
	    //TODO
	    //FIXME 特别注意，该接口不能被扫描到，否则会出错
	}
	
# 启动类

	@SpringBootApplication
	@MapperScan(basePackages = "cn.org.jetbrains.universalmapper.dao")
	public class UniversalmapperApplication {
	
		public static void main(String[] args) {
			SpringApplication.run(UniversalmapperApplication.class, args);
		}
	}
	
# Dao

	@Mapper
	public interface DemoDao  extends MyMapper<Demo>{
	}
