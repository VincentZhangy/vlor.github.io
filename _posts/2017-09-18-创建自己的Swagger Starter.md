---
layout:     post
title:      swagger-spring-boot-starter
subtitle:    "\"创建自己的Swagger Starter\""
date:       2017-09-18
author:     mrchang
header-img: img/post-bg-re-vs-ng2.jpg
catalog: true
tags:
    - Spring Boot 
    - Swagger
    - 接口文档
    - Starter

---


## 步骤

1. 新建maven工程，引入依赖
	
		<version.swagger>2.7.0</version.swagger>


		<dependency><!-- 以下两个依赖是自动配置的依赖 -->
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-autoconfigure</artifactId>
			<version>RELEASE</version>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-configuration-processor</artifactId>
			<version>RELEASE</version>
			<optional>true</optional>
		</dependency>
		<dependency>
			<groupId>io.springfox</groupId>
			<artifactId>springfox-swagger-ui</artifactId>
			<version>${version.swagger}</version>
		</dependency>
		<dependency>
			<groupId>io.springfox</groupId>
			<artifactId>springfox-swagger2</artifactId>
			<version>${version.swagger}</version>
		</dependency>
		
		
2. 创建说明配置类，省略get set方法...

		@ConfigurationProperties(prefix="swagger")
		public class SwaggerProperties {
		
		    /**是否开启swagger**/
		    private Boolean enabled;
		
		    private String basePackage = "";
		
		    private String title  = "Spring Boot中使用Swagger2构建RESTful APIs";
		
		    private String description = "更多Spring Boot相关文章请关注：https://jetbrains.org.cn/";
		
		    private String termsOfServiceUrl = "https://jetbrains.org.cn/" ;
		
		    private String contact ="Mr.Chang";
		
		    private String version = "1.0";
		    
3. 创建swagger配置

				@Configuration
				@ConditionalOnProperty(name = "swagger.enabled", matchIfMissing = true)
				@Import({
				        Swagger2DocumentationConfiguration.class})
				public class Swagger2Configuration {
				
				}
				

4. 创建自动配置类

		@Configuration
		@Import({
		        Swagger2Configuration.class
		})
		
		public class SwaggerAutoConfiguration  {
		    @Bean
		    @ConditionalOnMissingBean
		    public SwaggerProperties swaggerProperties() {
		        return new SwaggerProperties();
		    }
		
		
		    @Bean
		    @ConditionalOnMissingBean
		    @ConditionalOnProperty(name = "swagger.enabled", matchIfMissing = true)
		    public Docket createRestApi(SwaggerProperties swaggerProperties) {
		
		        return new Docket(DocumentationType.SWAGGER_2)
		                .apiInfo(new ApiInfoBuilder()
		                        .title(swaggerProperties.getTitle())
		                        .description(swaggerProperties.getDescription())
		                        .termsOfServiceUrl(swaggerProperties.getTermsOfServiceUrl())
		                        .contact(swaggerProperties.getContact())
		                        .version(swaggerProperties.getVersion())
		                        .build())
		                .select()
		                .apis(RequestHandlerSelectors.basePackage(swaggerProperties.getBasePackage()))
		                .paths(PathSelectors.any())
		                .build();
		    }
		}
		
		
5. 创建启动注解
	
		@Target({ElementType.TYPE})
		@Retention(RetentionPolicy.RUNTIME)
		@Documented
		@Inherited
		@Import({SwaggerAutoConfiguration.class})
		public @interface EnableSwagger2 {
		}
		
		
6. 使用
	* 第一次使用，将项目打包上传至私服，其他项目直接饮用，在springboot启动类上书写`@EnableSwagger2`即可
	* 非第一次使用。其他项目直接饮用，在springboot启动类上书写`@EnableSwagger2`即可
	
	