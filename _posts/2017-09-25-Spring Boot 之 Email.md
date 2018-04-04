---
layout:     post
title:     	Spring Boot 之 Email
subtitle:    "\"Spring Boot 之 Email\""
date:       2017-09-25
author:     Mr Chang
header-img: img/post-bg-2015.jpg
catalog: true
tags:
    - Spring Boot
    - Email

---


## POM

	<dependencies>
	        <dependency>
	            <groupId>org.springframework.boot</groupId>
	            <artifactId>spring-boot-starter-mail</artifactId>
	        </dependency>
	        <dependency>
	            <groupId>com.google.guava</groupId>
	            <artifactId>guava</artifactId>
	            <version>19.0</version>
	        </dependency>
	        <dependency>
	            <groupId>com.alibaba</groupId>
	            <artifactId>fastjson</artifactId>
	            <version>1.2.12</version>
	        </dependency>
	        <!-- common -->
	        <dependency>
	            <groupId>org.apache.commons</groupId>
	            <artifactId>commons-lang3</artifactId>
	            <version>3.2.1</version>
	        </dependency>
	</dependencies>
	 
	 
	
## application.properties

	# Email (MailProperties)
	spring.mail.default-encoding=UTF-8
	spring.mail.host=smtp.qq.com
	spring.mail.password=123456
	spring.mail.port=25
	spring.mail.protocol=smtp
	spring.mail.test-connection=false
	spring.mail.username=server1@qq.com
	#spring.mail.properties.mail.smtp.auth=true
	#spring.mail.properties.mail.smtp.starttls.enable=true
	#spring.mail.properties.mail.transport.protocol=smtps
	#spring.mail.properties.mail.smtps.quitwait=false
	
	

	
	
## EmailSender
		
	@Component("emailSender")
	public class EmailSender {
	    private Logger logger = LogManager.getLogger(getClass());
	    private String defaultFrom = "server1@qq.com";
	    @Autowired
	    private JavaMailSender javaMailSender;
	
	    /**
	     * 发送邮件
	     *
	     * @param to      收件人地址
	     * @param subject 邮件主题
	     * @param content 邮件内容
	     * @author lance
	     */
	    public boolean sender(String to, String subject, String content) {
	        return sender(to, subject, content, true);
	    }
	
	    /**
	     * 发送邮件
	     *
	     * @param to      收件人地址
	     * @param subject 邮件主题
	     * @param content 邮件内容
	     * @param html    是否格式内容为HTML
	     * @author lance
	     */
	    public boolean sender(String to, String subject, String content, boolean html) {
	        if (StringUtils.isBlank(to)) {
	            logger.error("邮件发送失败：收件人地址不能为空.");
	            return false;
	        }
	        return sender(new String[]{to}, subject, content, html);
	    }
	
	    /**
	     * sender message
	     *
	     * @param to
	     * @param subject
	     * @param content
	     * @param html
	     * @return
	     */
	    public boolean sender(String[] to, String subject, String content, boolean html) {
	        if (to == null || to.length == 0) {
	            logger.error("批量邮件发送失败：收件人地址不能为空.");
	            return false;
	        }
	
	        SimpleMailMessage simpleMailMessage = new SimpleMailMessage();
	        simpleMailMessage.setFrom(defaultFrom);
	        simpleMailMessage.setTo(to);
	        simpleMailMessage.setSubject(subject);
	        simpleMailMessage.setText(content);
	
	        try {
	            javaMailSender.send(simpleMailMessage);
	            return true;
	        } catch (MailException e) {
	            logger.error("发送邮件错误：{}, TO:{}, Subject:{},Content:{}.", e, to, subject, content);
	            return false;
	        }
	    }
	}

	
	
	    
	    
## 参考

**代码：https://github.com/changdaye/spring-boot-demo/tree/master/spring-boot-email**