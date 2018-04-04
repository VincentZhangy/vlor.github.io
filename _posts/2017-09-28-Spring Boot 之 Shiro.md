---
layout:     post
title:     	Spring Boot 之 Shiro
subtitle:    "\"Spring Boot 之 Shiro\""
date:       2017-09-27
author:     Mr Chang
header-img: img/post-bg-desk.jpg
catalog: true
tags:
    - Spring Boot
    - Shiro
    - 权限

---


## POM

	<dependency>
	            <groupId>org.apache.shiro</groupId>
	            <artifactId>shiro-core</artifactId>
	            <version>${shiro.version}</version>
	        </dependency>
	        <dependency>
	            <groupId>org.apache.shiro</groupId>
	            <artifactId>shiro-web</artifactId>
	            <version>${shiro.version}</version>
	        </dependency>
	
	        <dependency>
	            <groupId>org.apache.shiro</groupId>
	            <artifactId>shiro-spring</artifactId>
	            <version>${shiro.version}</version>
	 </dependency>
	 
	 
## ShiroConfig（主要代码）

	@Bean(name = "shiroFilter")
    public ShiroFilterFactoryBean shiroFilter() {
        System.out.println("ShiroConfiguration.shirFilter()");
        ShiroFilterFactoryBean shiroFilterFactoryBean = new ShiroFilterFactoryBean();
        shiroFilterFactoryBean.setSecurityManager(securityManager());
        //拦截器.
        Map<String, String> filterChainDefinitionMap = new LinkedHashMap<String, String>();
        // 配置不会被拦截的链接 顺序判断
        filterChainDefinitionMap.put("/static/**", "anon");
        filterChainDefinitionMap.put("/druid/**", "anon");
        //配置退出 过滤器,其中的具体的退出代码Shiro已经替我们实现了
        filterChainDefinitionMap.put("/logout", "logout");
        //过滤链定义，从上向下顺序执行，一般将/**放在最为下边
        // 如果不设置默认会自动寻找Web工程根目录下的"/login.jsp"页面
        shiroFilterFactoryBean.setLoginUrl("/login");
        // 登录成功后要跳转的链接
        shiroFilterFactoryBean.setSuccessUrl("/index");

        //未授权界面;
        shiroFilterFactoryBean.setUnauthorizedUrl("/403");
        //<!-- authc:所有url都必须认证通过才可以访问; anon:所有url都都可以匿名访问-->
        filterChainDefinitionMap.put("/**", "authc");


        shiroFilterFactoryBean.setFilterChainDefinitionMap(filterChainDefinitionMap);
        return shiroFilterFactoryBean;
    }
 
## URLPermissionsFilter（验证用户请求路径权限） 

	  @Override
	    public boolean isAccessAllowed(ServletRequest request, ServletResponse response, Object mappedValue) throws IOException {
	        String curUrl = getRequestUrl(request);
	        Subject subject = SecurityUtils.getSubject();
	        if (subject.getPrincipal() == null
	                || StringUtils.endsWithAny(curUrl, ".js", ".css", ".html")
	                || StringUtils.endsWithAny(curUrl, ".jpg", ".png", ".gif", ".jpeg")
	                || StringUtils.equals(curUrl, "/unauthor")) {
	            return true;
	        }
	        List<String> urls = userService.findPermissionUrl(subject.getPrincipal().toString());
	
	        return urls.contains(curUrl);
	    } 
	    
## UserRealm（登录用户验证）
	
	  //权限资源角色
	    @Override
	    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals) {
	        String username = (String) principals.getPrimaryPrincipal();
	        SimpleAuthorizationInfo info = new SimpleAuthorizationInfo();
	        //add Permission Resources
	        info.setStringPermissions(userService.findPermissions(username));
	        //add Roles String[Set<String> roles]
	        //info.setRoles(roles);
	        return info;
	    }
	
	    //登录验证
	    @Override
	    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
	        UsernamePasswordToken upt = (UsernamePasswordToken) token;
	        String userName = upt.getUsername();
	        UserInfo user = userService.findByAccount(userName);
	
	        if (user == null) {
	            throw new UnknownAccountException();
	        }
	        SimpleAuthenticationInfo info = new SimpleAuthenticationInfo(userName, user.getPassword(), getName());
	        return info;
	    } 

## LoginController（登录接口）

    @RequestMapping(value = "login", method = RequestMethod.POST)
    @ResponseBody
    public String login(HttpServletRequest request, RedirectAttributes rediect) {
        String account = request.getParameter("account");
        String password = request.getParameter("password");

        UsernamePasswordToken upt = new UsernamePasswordToken(account, password);
        Subject subject = SecurityUtils.getSubject();
        try {
            subject.login(upt);
        } catch (AuthenticationException e) {
            e.printStackTrace();
            rediect.addFlashAttribute("errorText", "您的账号或密码输入错误!");
            return "您的账号或密码输入错误";
        }
        return "登录成功";
    }


    @RequestMapping("unauthor")
    @ResponseBody
    public String unauthor() {
        return "没有权限";
    }
	    
## 参照

**代码：https://github.com/changdaye/spring-boot-shiro**

**参考：http://www.ityouknow.com/springboot/2017/06/26/springboot-shiro.html
**

 

