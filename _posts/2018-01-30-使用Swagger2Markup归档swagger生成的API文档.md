---
layout:     post
title:     	使用Swagger2Markup归档swagger生成的API文档
subtitle:    "\"离线存储 \""
date:       2018-01-30
author:     Mr Chang
header-img: img/post-bg-e2e-ux.jpg
catalog: true
tags:
    - Swagger2Markup
    - swagger

---


# 说明

项目中使用Swagger之后，我们能够很轻松的管理API文档，并非常简单的模拟接口调用，但是构建的文档必须通过在项目中整合 `swagger-ui`、或使用单独部署的 `swagger-ui`和 `/v2/api-docs`返回的配置信息才能展现出您所构建的API文档。本文将在使用Swagger的基础上，再介绍一种生成静态API文档的方法，以便于构建更轻量部署和使用的API文档。


# Swagger2Markup简介

Swagger2Markup是Github上的一个开源项目。该项目主要用来将Swagger自动生成的文档转换成几种流行的格式以便于静态部署和使用，比如：AsciiDoc、Markdown、Confluence。

项目主页：https://github.com/Swagger2Markup/swagger2markup

# 如何使用

在使用Swagger2Markup之前，我们先需要准备一个使用了Swagger的Web项目，可以是直接使用Swagger2的项目

 1. 引入依赖

 		<dependency>
			<groupId>io.github.swagger2markup</groupId>
			<artifactId>swagger2markup</artifactId>
			<version>1.3.1</version>
		</dependency>
		
		<repositories>
			<repository>
				<snapshots>
					<enabled>true</enabled>
					<updatePolicy>always</updatePolicy>
				</snapshots>
				<id>jcenter-releases</id>
				<name>jcenter</name>
				<url>http://jcenter.bintray.com</url>
			</repository>
		</repositories>
		
		
2. 编写一个单元测试用例来生成执行生成文档的代码

		@RunWith(SpringRunner.class)
		@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.DEFINED_PORT)
		public class SwaggerDemoApplicationTests {
		
			/**
			 * 生成AsciiDocs格式文档
			 * @throws Exception
			 */
			@Test
			public void generateAsciiDocs() throws Exception {
				//	输出Ascii格式
				Swagger2MarkupConfig config = new Swagger2MarkupConfigBuilder()
						.withMarkupLanguage(MarkupLanguage.ASCIIDOC)
						.withOutputLanguage(Language.ZH)
						.withPathsGroupedBy(GroupBy.TAGS)
						.withGeneratedExamples()
						.withoutInlineSchema()
						.build();
		
				Swagger2MarkupConverter.from(new URL("http://localhost:8080/v2/api-docs"))
						.withConfig(config)
						.build()
						.toFolder(Paths.get("./docs/asciidoc/generated"));
			}
		
			/**
			 * 生成Markdown格式文档
			 * @throws Exception
			 */
			@Test
			public void generateMarkdownDocs() throws Exception {
				//	输出Markdown格式
				Swagger2MarkupConfig config = new Swagger2MarkupConfigBuilder()
						.withMarkupLanguage(MarkupLanguage.MARKDOWN)
						.withOutputLanguage(Language.ZH)
						.withPathsGroupedBy(GroupBy.TAGS)
						.withGeneratedExamples()
						.withoutInlineSchema()
						.build();
		
				Swagger2MarkupConverter.from(new URL("http://localhost:8080/v2/api-docs"))
						.withConfig(config)
						.build()
						.toFolder(Paths.get("./docs/markdown/generated"));
			}
		
		
			/**
			 * 生成Confluence格式文档
			 * @throws Exception
			 */
			@Test
			public void generateConfluenceDocs() throws Exception {
				//	输出Confluence使用的格式
				Swagger2MarkupConfig config = new Swagger2MarkupConfigBuilder()
						.withMarkupLanguage(MarkupLanguage.CONFLUENCE_MARKUP)
						.withOutputLanguage(Language.ZH)
						.withPathsGroupedBy(GroupBy.TAGS)
						.withGeneratedExamples()
						.withoutInlineSchema()
						.build();
		
				Swagger2MarkupConverter.from(new URL("http://localhost:8080/v2/api-docs"))
						.withConfig(config)
						.build()
						.toFolder(Paths.get("./docs/confluence/generated"));
			}
		
			/**
			 * 生成AsciiDocs格式文档,并汇总成一个文件
			 * @throws Exception
			 */
			@Test
			public void generateAsciiDocsToFile() throws Exception {
				//	输出Ascii到单文件
				Swagger2MarkupConfig config = new Swagger2MarkupConfigBuilder()
						.withMarkupLanguage(MarkupLanguage.ASCIIDOC)
						.withOutputLanguage(Language.ZH)
						.withPathsGroupedBy(GroupBy.TAGS)
						.withGeneratedExamples()
						.withoutInlineSchema()
						.build();
		
				Swagger2MarkupConverter.from(new URL("http://localhost:8080/v2/api-docs"))
						.withConfig(config)
						.build()
						.toFile(Paths.get("./docs/asciidoc/generated/all"));
			}
		
			/**
			 * 生成Markdown格式文档,并汇总成一个文件
			 * @throws Exception
			 */
			@Test
			public void generateMarkdownDocsToFile() throws Exception {
				//	输出Markdown到单文件
				Swagger2MarkupConfig config = new Swagger2MarkupConfigBuilder()
						.withMarkupLanguage(MarkupLanguage.MARKDOWN)
						.withOutputLanguage(Language.ZH)
						.withPathsGroupedBy(GroupBy.TAGS)
						.withGeneratedExamples()
						.withoutInlineSchema()
						.build();
		
				Swagger2MarkupConverter.from(new URL("http://localhost:8080/v2/api-docs"))
						.withConfig(config)
						.build()
						.toFile(Paths.get("./docs/markdown/generated/all"));
		}



3. 使用maven插件生成HTML文档

		<plugins>
				<plugin>
					<groupId>org.asciidoctor</groupId>
					<artifactId>asciidoctor-maven-plugin</artifactId>
					<version>1.5.6</version>
					<configuration>
						<sourceDirectory>./docs/asciidoc/generated</sourceDirectory>
						<outputDirectory>./docs/asciidoc/html</outputDirectory>
						<headerFooter>true</headerFooter>
						<doctype>book</doctype>
						<backend>html</backend>
						<sourceHighlighter>coderay</sourceHighlighter>
						<attributes>
							<!--菜单栏在左边-->
							<toc>left</toc>
							<!--多标题排列-->
							<toclevels>3</toclevels>
							<!--自动打数字序号-->
							<sectnums>true</sectnums>
						</attributes>
					</configuration>
				</plugin>
		</plugins>
		
		
# 效果

![](http://cdn-blog.jetbrains.org.cn/18-1-30/77980397.jpg)

![](http://cdn-blog.jetbrains.org.cn/18-1-30/4926790.jpg)

### 实际项目应用后效果

* [实际项目应用后效果](http://cdn-blog.jetbrains.org.cn/doc/all.html）


# 代码

    https://github.com/changdaye/swagger-demo

# 鸣谢

   文章出处 ： http://blog.didispace.com/swagger2markup-asciidoc/





	


	