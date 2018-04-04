---
layout:     post
title:      Spring之CROS解决AJAX跨域问题
subtitle:    "\"Spring Boot之CROS解决AJAX跨域问题\""
date:       2017-09-22
author:     mrchang
header-img: img/post-bg-unix-linux.jpg
catalog: true
tags:
    - Spring Boot
    - CROS
    - AJAX
    - 跨域

---

## 说明

1. 出于安全考虑，浏览器禁止AJAX调用驻留在当前来源之外的资源。例如，当您在一个标签中检查您的银行帐户时，您可以将evil.com网站放在另一个标签中。evil.com的脚本不能使用您的凭据向您的银行API发出AJAX请求（从您的帐户中提款）！
2. [跨原始资源共享](https://spring.io/understanding/CORS)（CORS）是[大多数浏览器](http://caniuse.com/#feat=cors)实现的[W3C规范](https://www.w3.org/TR/cors/)，允许您以灵活的方式指定什么样的跨域请求被授权，而不是使用一些不太安全和不太强大的黑客，如IFrame或JSONP。
3. [Spring Framework 4.2 GA](https://spring.io/blog/2015/07/31/spring-framework-4-2-goes-ga)为开箱即用的[CORS](https://jira.spring.io/browse/SPR-9278)提供了[一流的支持](https://jira.spring.io/browse/SPR-9278)，为您提供了比典型的[基于过滤器的](http://software.dzhuvinov.com/cors-filter.html)解决方案更简单和更强大的配置方式。

## Spring MVC提供了高级配置功能

控制器方法CORS配置

* 您可以向`@RequestMapping`注释处理程序方法添加`@CrossOrigin`注释，以便启用CORS（默认情况下`@CrossOrigin`允许`@RequestMapping`注释中指定的所有起始和`HTTP`方法）：

		@RestController
		@RequestMapping("/account")
		public class AccountController {
		
			@CrossOrigin
			@GetMapping("/{id}")
			public Account retrieve(@PathVariable Long id) {
				// ...
			}
		
			@DeleteMapping("/{id}")
			public void remove(@PathVariable Long id) {
				// ...
			}
	}

* 也可以为整个控制器启用CORS：

		@CrossOrigin(origins = "http://domain2.com", maxAge = 3600)
		@RestController
		@RequestMapping("/account")
		public class AccountController {
		
			@GetMapping("/{id}")
			public Account retrieve(@PathVariable Long id) {
				// ...
			}
		
			@DeleteMapping("/{id}")
			public void remove(@PathVariable Long id) {
				// ...
			}
		}
		

在此示例中，对于这两种方法`retrieve()`和`remove()`处理程序方法都启用了CORS支持，还可以看到如何使用`@CrossOrigin`属性自定义CORS配置。

您甚至可以同时使用控制器和方法级CORS配置，然后Spring将组合两个注释属性来创建合并的CORS配置。

	@CrossOrigin(maxAge = 3600)
	@RestController
	@RequestMapping("/account")
	public class AccountController {
	
		@CrossOrigin(origins = "http://domain2.com")
		@GetMapping("/{id}")
		public Account retrieve(@PathVariable Long id) {
			// ...
		}
	
		@DeleteMapping("/{id}")
		public void remove(@PathVariable Long id) {
			// ...
		}
	}
	
如果您使用Spring Security，请确保在[Spring Security级别启用CORS](https://docs.spring.io/spring-security/site/docs/current/reference/html/cors.html)，并允许它利用Spring MVC级别定义的配置。

	@EnableWebSecurity
	public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
	
		@Override
		protected void configure(HttpSecurity http) throws Exception {
			http.cors().and()...
		}
	}
	

## 全局CORS配置

除了细粒度的基于注释的配置，您也可能想要定义一些全局CORS配置。这与使用过滤器类似，但可以使用Spring MVC声明并结合细粒度`@CrossOrigin`配置。默认情况下，所有的起源和`GET`，`HEAD`和`POST`方法都是允许的。


###JavaConfig

为整个应用程序启用CORS类似于

		@Configuration
		@EnableWebMvc
		public class WebConfig extends WebMvcConfigurerAdapter {
		
			@Override
			public void addCorsMappings(CorsRegistry registry) {
				registry.addMapping("/**");
			}
		}

如果您使用的是Spring Boot，建议只要声明一个`WebMvcConfigurer`bean如下：

	@Configuration
	public class MyConfiguration {
	
	    @Bean
	    public WebMvcConfigurer corsConfigurer() {
	        return new WebMvcConfigurerAdapter() {
	            @Override
	            public void addCorsMappings(CorsRegistry registry) {
	                registry.addMapping("/**");
	            }
	        };
	    }
	}		
您可以轻松地更改任何属性，以及仅将此CORS配置应用于特定路径模式：

	@Override
	public void addCorsMappings(CorsRegistry registry) {
		registry.addMapping("/api/**")
			.allowedOrigins("http://domain2.com")
			.allowedMethods("PUT", "DELETE")
				.allowedHeaders("header1", "header2", "header3")
			.exposedHeaders("header1", "header2")
			.allowCredentials(false).maxAge(3600);
	}
	
如果您使用Spring Security，请确保在[Spring Security级别启用CORS](https://docs.spring.io/spring-security/site/docs/current/reference/html/cors.html)，并允许它利用Spring MVC级别定义的配置。

###XML命名空间

也可以使用[mvc XML命名空间](https://jira.spring.io/browse/SPR-13046)配置CORS 。

这种最小的XML配置使得`/**`路径模式上的CORS 具有与JavaConfig相同的默认属性：

	<mvc:cors>
		<mvc:mapping path="/**" />
	</mvc:cors>

也可以使用自定义属性声明几个CORS映射：

	<mvc:cors>
	
		<mvc:mapping path="/api/**"
			allowed-origins="http://domain1.com, http://domain2.com"
			allowed-methods="GET, PUT"
			allowed-headers="header1, header2, header3"
			exposed-headers="header1, header2" allow-credentials="false"
			max-age="123" />
	
		<mvc:mapping path="/resources/**"
			allowed-origins="http://domain1.com" />
	
	</mvc:cors>
	
如果您使用`Spring Security`，请不要忘记在[`Spring Security`级别`启用CORS`](https://docs.spring.io/spring-security/site/docs/current/reference/html/cors.html)：

	<http>
		<!-- Default to Spring MVC's CORS configuration -->
		<cors />
		...
	</http>
	
##它是如何工作的？

CORS请求（[包括带有`OPTIONS`方法的预检](https://github.com/spring-projects/spring-framework/blob/master/spring-webmvc/src/main/java/org/springframework/web/servlet/FrameworkServlet.java#L906)）请求被自动发送到已`HandlerMapping`注册的各种。他们处理CORS预检要求和拦截CORS简单而实际的请求得益于[CorsProcessor](https://docs.spring.io/spring/docs/4.2.x/javadoc-api/org/springframework/web/cors/CorsProcessor.html)实现（[DefaultCorsProcessor](https://github.com/spring-projects/spring-framework/blob/master/spring-web/src/main/java/org/springframework/web/cors/DefaultCorsProcessor.java)以添加相关CORS响应头（如默认情况下）`Access-Control-Allow-Origin`）。[CorsConfiguration](https://docs.spring.io/spring/docs/4.2.x/javadoc-api/org/springframework/web/cors/CorsConfiguration.html)允许您指定如何处理CORS请求：允许的起点，头，方法等。它可以以各种方式提供：

1. `AbstractHandlerMapping#setCorsConfiguration()`允许在路径模式上映射`Map`几个[CorsConfiguration](https://docs.spring.io/spring/docs/4.2.x/javadoc-api/org/springframework/web/cors/CorsConfiguration.html)来指定一个`/api/**`
2. 子类可以`CorsConfiguration`通过重写`AbstractHandlerMapping#getCorsConfiguration(Object, HttpServletRequest)`方法提供自己的子类
3. 处理程序可以实现`CorsConfigurationSource`接口（像`ResourceHttpRequestHandler`现在这样），以便为每个请求提供[CorsConfiguration](https://docs.spring.io/spring/docs/4.2.x/javadoc-api/org/springframework/web/cors/CorsConfiguration.html)。

## 基于过滤器的CORS支持

作为上述其他方法的替代方法，`Spring Framework`还提供了一个[CorsFilter](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/filter/CorsFilter.html)。在这种情况下，而不是使用`@CrossOrigin`或者`WebMvcConfigurer#addCorsMappings(CorsRegistry)`您可以例如在Spring Boot应用程序中声明过滤器如下：

	@Configuration
	public class MyConfiguration {
	
		@Bean
		public FilterRegistrationBean corsFilter() {
			UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
			CorsConfiguration config = new CorsConfiguration();
			config.setAllowCredentials(true);
			config.addAllowedOrigin("http://domain1.com");
			config.addAllowedHeader("*");
			config.addAllowedMethod("*");
			source.registerCorsConfiguration("/**", config);
			FilterRegistrationBean bean = new FilterRegistrationBean(new CorsFilter(source));
			bean.setOrder(0);
			return bean;
		}
	}


	
	
