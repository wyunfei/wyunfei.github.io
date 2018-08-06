---
layout: post
title: 'SpringBoot shiro'
date: 2017-11-12
author: yunfei
tags: SpringBoot shiro
---

### 1、引入依赖
````xml
      <dependency>
         <groupId>org.apache.shiro</groupId>
         <artifactId>shiro-spring</artifactId>
         <version>1.4.0</version>
      </dependency>
````

### 2、代码配置

#### （1）DemoShiroRealm.java
````java
import com.demo.entity.SysUser;
import com.demo.service.SysResourceService;
import org.apache.shiro.authc.AccountException;
import org.apache.shiro.authc.AuthenticationException;
import org.apache.shiro.authc.AuthenticationInfo;
import org.apache.shiro.authc.AuthenticationToken;
import org.apache.shiro.authc.LockedAccountException;
import org.apache.shiro.authc.SimpleAuthenticationInfo;
import org.apache.shiro.authz.AuthorizationInfo;
import org.apache.shiro.authz.SimpleAuthorizationInfo;
import org.apache.shiro.realm.AuthorizingRealm;
import org.apache.shiro.subject.PrincipalCollection;
import org.apache.shiro.util.ByteSource;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
 
import com.comba.service.SysUserService;
import java.util.Set;
 
public class DemoShiroRealm extends AuthorizingRealm {
   private Logger logger = LoggerFactory.getLogger(CombaShiroRealm.class);
 
   @Autowired
   private SysUserService userService;
 
   @Autowired
   private SysResourceService sysResourceService;
 
   @Override
   protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals) {
      logger.info("权限配置-->MyShiroRealm.doGetAuthorizationInfo()");
      SimpleAuthorizationInfo authorizationInfo = new SimpleAuthorizationInfo();
      SysUser user = (SysUser) principals.getPrimaryPrincipal();
      Set<String> perms = sysResourceService.userAuthority(user.getId());
      logger.info("perms>{}",perms);
      authorizationInfo.setStringPermissions(perms);
 
      return authorizationInfo;
   }
 
   /* 主要是用来进行身份认证的，也就是说验证用户输入的账号和密码是否正确。 */
   @Override
   protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
      logger.info("MyShiroRealm.doGetAuthenticationInfo()");
      // 获取用户的输入的账号.
      String userName = (String) token.getPrincipal();
      logger.info("userName>{}", userName);
      // 通过username从数据库中查找 User对象，如果找到，没找到.
      // 实际项目中，这里可以根据实际情况做缓存，如果不做，Shiro自己也是有时间间隔机制，2分钟内不会重复执行该方法
      SysUser user = new SysUser();
      user.setUserName(userName);
      user = userService.findByUserName(user);
      if (user == null) {
         throw new AccountException("帐号或密码不正确！");
      }
      // 帐号锁定或已失效
      if (!"0".equals(user.getStatus())) {
         throw new LockedAccountException("账号已被禁用,请联系管理员");
      }
      SimpleAuthenticationInfo authenticationInfo = new SimpleAuthenticationInfo(user, // 用户名
            user.getPassword(), // 密码
            ByteSource.Util.bytes(user.getCredentialsSalt()), // salt=username+salt
            getName()// realm name
      );
      return authenticationInfo;
   }
}
````
#### （2）ShiroConfig.java
````java
import at.pollux.thymeleaf.shiro.dialect.ShiroDialect;
import org.apache.shiro.authc.credential.HashedCredentialsMatcher;
import org.apache.shiro.codec.Base64;
import org.apache.shiro.mgt.SecurityManager;
import org.apache.shiro.spring.security.interceptor.AuthorizationAttributeSourceAdvisor;
import org.apache.shiro.spring.web.ShiroFilterFactoryBean;
import org.apache.shiro.web.mgt.CookieRememberMeManager;
import org.apache.shiro.web.mgt.DefaultWebSecurityManager;
import org.apache.shiro.web.servlet.SimpleCookie;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.handler.SimpleMappingExceptionResolver;
import java.util.LinkedHashMap;
import java.util.Map;
import java.util.Properties;
 
@Configuration
public class ShiroConfig {
   private Logger logger = LoggerFactory.getLogger(ShiroConfig.class);
 
   @Bean
   public ShiroFilterFactoryBean shirFilter(SecurityManager securityManager) {
      logger.info("ShiroConfiguration.shirFilter()");
      ShiroFilterFactoryBean shiroFilterFactoryBean = new ShiroFilterFactoryBean();
      shiroFilterFactoryBean.setSecurityManager(securityManager);
      // 拦截器.
      Map<String, String> filterChainDefinitionMap = new LinkedHashMap<String, String>();
      // 配置不会被拦截的链接 顺序判断
      // <!-- authc:所有url都必须认证通过才可以访问; anon:所有url都都可以匿名访问-->
      filterChainDefinitionMap.put("/static/**", "anon");
      filterChainDefinitionMap.put("/favicon.ico", "anon");
      filterChainDefinitionMap.put("/assets/**", "anon");
      filterChainDefinitionMap.put("/js/**", "anon");
      filterChainDefinitionMap.put("/css/**", "anon");
      // 配置退出 过滤器,其中的具体的退出代码Shiro已经替我们实现了
      filterChainDefinitionMap.put("/logout", "anon");
      // 不拦截mqtt调用
      filterChainDefinitionMap.put("/*/processReport", "anon");
      // <!-- 过滤链定义，从上向下顺序执行，一般将/**放在最为下边 -->:这是一个坑呢，一不小心代码就不好使了;
      // <!-- authc:所有url都必须认证通过才可以访问; anon:所有url都都可以匿名访问-->
      filterChainDefinitionMap.put("/**", "authc");
      // 如果不设置默认会自动寻找Web工程根目录下的"/login.jsp"页面
      shiroFilterFactoryBean.setLoginUrl("/login");
      // 登录成功后要跳转的链接
      shiroFilterFactoryBean.setSuccessUrl("/product/index");
 
      // 配置记住我或认证通过可以访问的地址
      filterChainDefinitionMap.put("/product/index", "user"); // /**
 
      // 未授权界面;
      shiroFilterFactoryBean.setUnauthorizedUrl("/403");
      shiroFilterFactoryBean.setFilterChainDefinitionMap(filterChainDefinitionMap);
      logger.info("filterChainDefinitionMap>{}", filterChainDefinitionMap);
      return shiroFilterFactoryBean;
   }
 
   /**
    * 凭证匹配器 （由于我们的密码校验交给Shiro的SimpleAuthenticationInfo进行处理了 ）
    * 
    * @return
    */
   @Bean
   public HashedCredentialsMatcher hashedCredentialsMatcher() {
      HashedCredentialsMatcher hashedCredentialsMatcher = new HashedCredentialsMatcher();
      hashedCredentialsMatcher.setHashAlgorithmName("md5");// 散列算法:这里使用MD5算法;
      hashedCredentialsMatcher.setHashIterations(2);// 散列的次数，比如散列两次，相当于md5(md5(""));
      hashedCredentialsMatcher.setStoredCredentialsHexEncoded(true); //如果是true就使用realm所加密的hex 否则就使用base64 hex
      return hashedCredentialsMatcher;
   }
 
   @Bean
   public CombaShiroRealm myShiroRealm() {
      CombaShiroRealm myShiroRealm = new CombaShiroRealm();
      myShiroRealm.setCredentialsMatcher(hashedCredentialsMatcher());
      return myShiroRealm;
   }
 
   @Bean
   public ShiroDialect shiroDialect() {
      return new ShiroDialect();
   }
 
   @Bean
   public SecurityManager securityManager() {
      DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
      securityManager.setRealm(myShiroRealm());
      // 注入记住我管理器;
      securityManager.setRememberMeManager(rememberMeManager());
      return securityManager;
   }
 
  
 
   /**
    * 开启shiro aop注解支持. 使用代理方式;所以需要开启代码支持;
    * 
    * @param securityManager
    * @return
    */
   @Bean
   public AuthorizationAttributeSourceAdvisor authorizationAttributeSourceAdvisor(SecurityManager securityManager) {
      AuthorizationAttributeSourceAdvisor authorizationAttributeSourceAdvisor = new AuthorizationAttributeSourceAdvisor();
      authorizationAttributeSourceAdvisor.setSecurityManager(securityManager);
      return authorizationAttributeSourceAdvisor;
   }
 
   @Bean(name = "simpleMappingExceptionResolver")
   public SimpleMappingExceptionResolver createSimpleMappingExceptionResolver() {
      SimpleMappingExceptionResolver r = new SimpleMappingExceptionResolver();
      Properties mappings = new Properties();
      mappings.setProperty("DatabaseException", "databaseError");// 数据库异常处理
      mappings.setProperty("UnauthorizedException", "403");
      r.setExceptionMappings(mappings); // None by default
      r.setDefaultErrorView("error"); // No default
      r.setExceptionAttribute("ex"); // Default is "exception"
      // r.setWarnLogCategory("example.MvcLogger"); // No default
      return r;
   }
 
   /**
    * cookie对象;
    * 
    * @return
    */
   @Bean
   public SimpleCookie rememberMeCookie() {
      logger.info("ShiroConfiguration.rememberMeCookie()");
      // 这个参数是cookie的名称，对应前端的checkbox的name = rememberMe
      SimpleCookie simpleCookie = new SimpleCookie("rememberMe");
      // <!-- 记住我cookie生效时间3天 259200,单位秒;-->
      simpleCookie.setMaxAge(259200);
      return simpleCookie;
   }
 
   /**
    * cookie管理对象;
    * 
    * @return
    */
   @Bean
   public CookieRememberMeManager rememberMeManager() {
      logger.info("ShiroConfiguration.rememberMeManager()");
      CookieRememberMeManager cookieRememberMeManager = new CookieRememberMeManager();
      cookieRememberMeManager.setCookie(rememberMeCookie());
      //rememberMe cookie加密的密钥 建议每个项目都不一样 默认AES算法 密钥长度(128 256 512 位)
      cookieRememberMeManager.setCipherKey(Base64.decode("2AvVhdsgUs0FSA3SDFAdag=="));
      return cookieRememberMeManager;
   }
}
````
### 3、shiro 密码加密
http://jinnianshilongnian.iteye.com/blog/2021439
https://github.com/zhangkaitao/shiro-example
````java
 @RestController
 public class DemoController{
       @RequiresPermissions("user:create")
       @RequestMapping(value = "/save")
       public ModelMap save(SysUser user) {
          String salt = new SecureRandomNumberGenerator().nextBytes().toHex();
          user.setId("1");
          user.setName("admin");
          user.setSystemId("1");
          user.setUsername("admin");
          user.setSalt(salt);
          String password = "admin";
          //散列两次，相当于md5(md5(""));user.getCredentialsSalt:username+salt
          String newPwd = new Md5Hash(password, user.getCredentialsSalt(), 2).toString();
          logger.info("password>{}", newPwd);
          user.setPassword(newPwd);
          user.setStatus("0");
          user.setSex("1");
          ModelMap result = new ModelMap();
          String msg = user.getUsername() == null ? "新增成功!" : "更新成功!";
          userService.save(user);
          result.put("user", user);
          result.put("msg", msg);
          return result;
       }
 }
````
### 4、springboot thymeleaf和shiro标签整合
https://www.cnblogs.com/xiaojf/p/6613537.html
#### （1）添加依赖
````xml
<dependency>
   <groupId>com.github.theborakompanioni</groupId>
   <artifactId>thymeleaf-extras-shiro</artifactId>
   <version>1.2.1</version>
</dependency>
````
#### （2）在shiro的configuration中配置
````java
@Bean
public ShiroDialect shiroDialect() {
   return new ShiroDialect();
}​​​​​​​
````

#### （3）在html中加入xmlns
```html
<html xmlns:th="http://www.thymeleaf.org" xmlns:shiro="http://www.pollix.at/thymeleaf/shiro">

<button shiro:hasPermission="sys:user:addOrEdit" data-action="add" class="layui-btn layui-btn-normal">
    <i class="layui-icon" aria-hidden="true">&#xe608;</i> 新增</button>
```
![image.png](/assets/img/shiro.png)
