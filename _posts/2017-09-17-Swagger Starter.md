
---
layout:     post
title:      Swagger 使用
subtitle:    "\"Spring Boot 项目集成Swagger管理接口文档\""
date:       2017-09-17
author:     mrchang
header-img: img/post-bg-re-vs-ng2.jpg
catalog: true
tags:
    - Spring Boot 
    - Swagger
    - 接口文档

---


## 步骤

1. 创建一个SpringBoot - Web项目 ，pom中依赖坐标如下图所示

	![](http://cdn-blog.jetbrains.org.cn/17-9-17/91456957.jpg)
	
2. 引入Swagger 依赖

	![](http://cdn-blog.jetbrains.org.cn/17-9-17/33719682.jpg)
	
3. 在SpringBootStarterApplication同级目录创建Swagger配置类
	
	![](http://cdn-blog.jetbrains.org.cn/17-9-17/73059632.jpg)
	
4. 修改配置只需要在application.properties中修改自己的配置即可。我的项目是Spring Initializr自动构建的，如你的项目不是，那配置文件名字可能是 application-XXX.yml一类的。

5. Controller中使用
	![](http://cdn-blog.jetbrains.org.cn/17-9-17/6005778.jpg)

6. 文档效果

    ![](http://cdn-blog.jetbrains.org.cn/17-9-17/63184256.jpg)
	
7. 生产环境一般不会暴露接口文档出来，记得关闭开关。

	![](http://cdn-blog.jetbrains.org.cn/17-9-17/59638105.jpg)
	
## API的注解

对于API的设计，一般倾向于将功能相同的API归集为一组。在Spring Boot中，利用Controller来实现，每个Controller里包含若干个REST API，而每个API都有输入及输出值。所以swagger对API的注解也是参照这个层级来划分与实现的。其逻辑结果如下图：
	![](http://cdn-blog.jetbrains.org.cn/17-9-17/67200541.jpg)
	
1. @Api
该注解将一个Controller（Class）标注为一个swagger资源（API）。在默认情况下，Swagger-Core只会扫描解析具有@Api注解的类，而会自动忽略其他类别资源（JAX-RS endpoints，Servlets等等）的注解。该注解包含以下几个重要属性：
	* tags
		
		API分组标签。具有相同标签的API将会被归并在一组内展示。
	* value
		如果tags没有定义，value将作为Api的tags使用
		
	* description
		API的详细描述，在1.5.X版本之后不再使用，但实际发现在2.0.0版本中仍然可以使用
		
2. @ApiOperation
在指定的（路由）路径上，对一个操作或HTTP方法进行描述。具有相同路径的不同操作会被归组为同一个操作对象。不同的HTTP请求方法及路径组合构成一个唯一操作。此注解的属性有：
	* value
		对操作的简单说明，长度为120个字母，60个汉字。
		
	* notes
		对操作的详细说明。
		
	* httpMethod
		HTTP请求的动作名，可选值有："GET", "HEAD", "POST", "PUT", "DELETE", "OPTIONS" and "PATCH"。
		
	* code
		默认为200，有效值必须符合标准的HTTP Status Code Definitions。
		
3. @ApiImplicitParams
	注解ApiImplicitParam的容器类，以数组方式存储。
	
4. @ApiImplicitParam
对API的单一参数进行注解。虽然注解@ApiParam同JAX-RS参数相绑定，但这个@ApiImplicitParam注解可以以统一的方式定义参数列表，也是在Servelet及非JAX-RS环境下，唯一的方式参数定义方式。注意这个注解@ApiImplicitParam必须被包含在注解@ApiImplicitParams之内。可以设置以下重要参数属性：
	* name
		参数名称
		
	* value
		参数的简短描述
		
	* required
		是否为必传参数
		
	* dataType
		参数类型，可以为类名，也可以为基本类型（String，int、boolean等）
		
	* paramType
		参数的传入（请求）类型，可选的值有path, query, body, header or form。
		
5. @ApiParam
	增加对参数的元信息说明。这个注解只能被使用在JAX-RS 1.x/2.x的综合环境下。其主要的属性有：
	* required
		是否为必传参数
		
	* value
		参数简短说明
		
6. @ApiResponses
	注解@ApiResponse的包装类，数组结构。即使需要使用一个@ApiResponse注解，也需要将@ApiResponse注解包含在注解@ApiResponses内。
	
7. @ApiResponse
描述一个操作可能的返回结果。当REST API请求发生时，这个注解可用于描述所有可能的成功与错误码。可以用，也可以不用这个注解去描述操作的返回类型，但成功操作的返回类型必须在@ApiOperation中定义。如果API具有不同的返回类型，那么需要分别定义返回值，并将返回类型进行关联。但Swagger不支持同一返回码，多种返回类型的注解。注意：这个注解必须被包含在@ApiResponses注解中。
	* code
		HTTP请求返回码。有效值必须符合标准的HTTP Status Code Definitions。
		
	* message
		更加易于理解的文本消息
		
	* response
		返回类型信息，必须使用完全限定类名，比如“com.xyz.cc.Person.class”。
		
	* responseContainer
		如果返回类型为容器类型，可以设置相应的值。有效值为 "List", "Set" or "Map"，其他任何无效的值都会被忽略。

# Model的注解

对于Model的注解，Swagger提供了两个：@ApiModel及@ApiModelProperty，分别用以描述Model及Model内的属性。

1. @ApiModel
提供对Swagger model额外信息的描述。在标注@ApiOperation注解的操作内，所有的类将自动被内省（introspected），但利用这个注解可以做一些更加详细的model结构说明。主要属性有：
	
	* value
		model的别名，默认为类名
		
	* description
		model的详细描述
	
2. @ApiModelProperty
	对model属性的注解，主要的属性值有：
	
	* value
		属性简短描述
		
	* example
		属性的示例值
		
	* required
		是否为必须值
	
## 总结

   `@ApiImplicitParams`与`@ApiModelProperty`搭配使用更灵活
   
## 关于Token问题

考虑到安全的问题，每次请求API需要对用户进行验证与授权。目前主流的验证方式采用请求头部（request header）传递token，即用户登录之后获取一个token，然后每次都使用这个token去请求API。如果想利用swagger-UI进行API测试，必须显式为每个需要验证的API指定token参数。这时可以为每个操作添加一个注解@ApiImplicitParams，具体代码如下：

    @ApiImplicitParams({@ApiImplicitParam(name = "TOKEN", value = "Authorization token", required = true, dataType = "string", paramType = "header")})

