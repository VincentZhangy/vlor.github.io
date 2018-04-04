---
layout:     post
title:     	IntelliJ IDEA 使用 lombok
subtitle:    "\"减少冗长代码，使pojo更简洁\""
date:       2017-09-27
author:     Mr Chang
header-img: img/post-bg-cook.jpg
catalog: true
tags:
    - IntelliJ IDEA
    - lombok

---

对于 POJO, 我们需要为其中的每个字段生成 getter,setter 方法, 虽然能够使用 IDE 快速为我们生成. 但如果需要修改字段名称及字段类型, 那么就需要删除在重新进行生成, 终究还是有一些不方便. 如果使用 lombok, 可以通过一些简单的注解直接生成我们所需要的代码, 能极大的提高开发体验.

1. pom引入依赖

		<dependency>
		      <groupId>org.projectlombok</groupId>
		      <artifactId>lombok</artifactId>
		      <optional>true</optional>
		 </dependency>
		 
2. idea下载插件

	![](http://cdn-blog.jetbrains.org.cn/17-9-27/28118098.jpg)
	
3. lombok 常用注解介绍

	* `@NonNull` ： 使用 @NonNull 注解修饰的字段 通过 set 方法设置时如果为 null, 将抛出 NullPointerException
	* `@Cleanup` ： 主要用来修饰 IO 流相关类, 会在 finally 代码块中对该资源进行 close();
	* `@Getter,@Setter` ： 为字段生成 getter,setter 方法, 标记到类上表明为所有字段生成
	* `@ToString` ： 生成 toString 方法, 默认打印所有非静态字段
	* `@EqualsAndHashCode` ： 生成 equals 和 hashCode 方法
	* `@NoArgsConstructor` ： 无参构造函数
	* `@RequiredArgsConstructor` ： 为未初始化的 final 字段和使用 @NonNull 标注的字段生成构造函数
	* `@AllArgsConstructor` ： 为所有字段生成构造函数
	* `@Data` ： 相当于同时使@Getter,@Setter,@ToString,@EqualsAndHashCode,@RequiredArgsConstructor
	* `@Value` ： 类将使用 final 进行修饰,同时使用@ToString,@EqualsAndHashCode,@AllArgsConstructor,@Getter
	* `@Builder` ： 创建一个静态内部类, 使用该类可以使用链式调用创建对象
如 User 对象中存在 name,age 字段, User user=User.builder().name("姓名").age(20).build()
	* `@SneakyThrows` ： 对标注的方法进行 try catch 后抛出异常, 可在 value 输入需要 catch 的异常数组, 默认 catch Throwable
	* `@Synchronized` ： 在标注的方法内 使用 synchronized(\$lock) {} 对代码进行包裹 ,$lock 为 new Object[0]
	* `@Log,@CommonsLog,@JBossLog,@Log,@Log4j,@Log4j2,@Slf4j,@XSlf4j` ： 生成一个当前类的日志对象, 可以使用 topic 指定要获取的日志名称,使用log...使用

4. 自定义配置
	
	虽然 lombok 能为我们快速生成代码, 但是有一些生成的代码依然无法满足我们的需求. 此时可配置 lombok.config 来解决问题
	
	以下列出一些常用的配置
	
		 lombok.getter.noIsPrefix=true(默认: false)  #lombok 默认对 boolean 类型字段生成的 get 方法使用 is 前缀, 通过此配置则使用 get 前缀
		 lombok.accessors.chain=true(默认: false) #默认的 set 方法返回 void 设置为 true 返回调用对象本身
		 lombok.accessors.fluent=true(默认: false) #如果设置为 true. get,set 方法将不带 get,set 前缀, 直接以字段名为方法名
		 lombok.log.fieldName=logger(默认: log) #设置 log 类注解返回的字段名称

**注 : 在 IDEA 中,lombok.config 文件 请放置于 src\main\java 目录下, 在 src\main\resources 中将不生效**



![](http://cdn-blog.jetbrains.org.cn/17-10-11/92200804.jpg)