# Shiro权限框架学习笔记

## 简介
• Apache Shiro 是 Java  的一个安全（权限）框架。 • Shiro 可以非常容易的开发出足够好的应用，其不仅可以用在 JavaSE 环境，也可以用在 JavaEE 环境。 • Shiro 可以完成：认证、授权、加密、会话管理、与Web 集成、缓存 等。 

• 下载：http://shiro.apache.org/

### 功能简介 

• 基本功能点如下图所示：

![image-20200716220247452](C:\Users\XXX\AppData\Roaming\Typora\typora-user-images\image-20200716220247452.png)



1、 Authentication 认证 ---- 用户登录

2、 Authorization 授权 --- 用户具有哪些权限

3、 Cryptography 安全数据加密 

4、 Session Management 会话管理 

5、 Web Integration web系统集成 

6、 Interations 集成其它应用，spring、缓存框架

## Shiro的核心API

Subject： 用户主体（把操作交给SecurityManager）

SecurityManager：安全管理器（关联Realm）

Realm：Shiro连接数据的桥梁

### 导入shiro与spring整合依赖

修改pom.xml

```xml
<!-- shiro与spring整合依赖 -->		
<dependency>
	<groupId>org.apache.shiro</groupId>
    <artifactId>shiro-spring</artifactId>
    <version>1.5.3</version>
</dependency>
```

### 自定义Realm类

```java
package com.ttt.shiro;

import org.apache.shiro.authc.AuthenticationException;
import org.apache.shiro.authc.AuthenticationInfo;
import org.apache.shiro.authc.AuthenticationToken;
import org.apache.shiro.authz.AuthorizationInfo;
import org.apache.shiro.realm.AuthorizingRealm;
import org.apache.shiro.subject.PrincipalCollection;

/**
 * 自定义Realm
 */
public class UserRealm extends AuthorizingRealm {
    /**
     * 授权逻辑
     * @param principalCollection
     * @return
     */
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
        System.out.println("执行授权逻辑");
        return null;
    }

    /**
     * 认证逻辑
     * @param authenticationToken
     * @return
     * @throws AuthenticationException
     */
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {
        System.out.println("执行认证逻辑");
        return null;
    }
}
```

### 编写Shiro配置类

```java
package com.ttt.shiro;

import org.apache.shiro.spring.web.ShiroFilterFactoryBean;
import org.apache.shiro.web.mgt.DefaultWebSecurityManager;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * shiro配置类
 */
@Configuration
public class ShiroConfig {
    /**
     * 创建ShiroFilterFactoryBean
     * @return 创建
     */
    public ShiroFilterFactoryBean getShiroFilterFactoryBean(@Qualifier("securityManager")DefaultWebSecurityManager securityManager) {
        ShiroFilterFactoryBean shiroFilterFactoryBean = new ShiroFilterFactoryBean();
        //设置安全管理器
        shiroFilterFactoryBean.setSecurityManager(securityManager);
        return shiroFilterFactoryBean;
    }

    /**
     * 创建DefaultWebSecurityManager
     * @return 创建
     */
    @Bean("securityManager")
    public DefaultWebSecurityManager getDefaultWebSessionManager(@Qualifier("UserRealm") UserRealm userRealm) {
        DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
        securityManager.setRealm(userRealm);
        return securityManager;

    }

    /**
     * 创建Realm
     * 放入spring环境
     *
     * @return 创建
     */
    @Bean("UserRealm")
    public UserRealm getRealm() {
        return new UserRealm();
    }
}
```

## shiro内置过滤器

### 添加Shiro内置过滤器

Shiro内置过滤器，可以实现权限相关的拦截器

 常用的过滤器：

		 *    anon: 无需认证（登录）可以访问

		 *    authc: 必须认证才可以访问

		 *    user: 如果使用rememberMe的功能可以直接访问

		 *    perms： 该资源必须得到资源权限才可以访问

   * role: 该资源必须得到角色权限才可以访问

      

在配置类中

```java
/**
     * 创建ShiroFilterFactoryBean
     * @return 创建
     */
    @Bean
    public ShiroFilterFactoryBean getShiroFilterFactoryBean(@Qualifier("securityManager")DefaultWebSecurityManager securityManager) {
        ShiroFilterFactoryBean shiroFilterFactoryBean = new ShiroFilterFactoryBean();
        //设置安全管理器
        shiroFilterFactoryBean.setSecurityManager(securityManager);
        Map<String, String> filterMap = new LinkedHashMap<String, String>();
        //第一种方法
        /*filterMap.put("/add", "authc");
        filterMap.put("/update", "authc");*/
        //第二种方法
        filterMap.put("/ThymeleafDemo", "anon");
        filterMap.put("/*", "authc");
        //设置登录方法
        shiroFilterFactoryBean.setLoginUrl("/toLogin");
        shiroFilterFactoryBean.setFilterChainDefinitionMap(filterMap);
        return shiroFilterFactoryBean;
    }
```

## 实例

### 实现登陆操作

### 前端页面设计

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.w3.org/1999/xhtml">
<head>
    <meta charset="UTF-8">
    <title>登录</title>
</head>
<body>
    <h1>登陆页面</h1>
    <h3 style="color: red" th:text="${msg}"></h3>
<form method="post" action="login">
    <table>
        <tr>
            <td>用户名:</td>
            <td><input type="text" name="name"></td>
        </tr>
        <tr>
            <td>密码:</td>
            <td><input type="password" name="password"></td>
        </tr>
        <tr>
            <td><input type="submit" value="登录"></td>
            <td><input type="reset" value="重置"></td>
        </tr>
    </table>
</form>
</body>
</html>
```

### 登录逻辑处理

使用Shiro编写认证操作

1. 获取Subject

2. 封装用户数据

3. 执行登录方法

```java
@RequestMapping("login")
    public String Login(String name,String password,Model model) {
        System.out.println("成功进入登录验证页面");

        Subject subject = SecurityUtils.getSubject();
        UsernamePasswordToken token = new UsernamePasswordToken(name, password);
        try {
            subject.login(token);
            //登陆成功
            return "redirect:ThymeleafDemo";
        } catch (UnknownAccountException e) {
            System.out.println("用户名不存在");
            model.addAttribute("msg", "用户名不存在");
            return "login";
        } catch (IncorrectCredentialsException e) {
            model.addAttribute("msg", "密码错误");
            return "login";
        }
    }
```

### 编写Reaml得判断逻辑

```java
    /**
     * 认证逻辑
     * @param authenticationToken
     * @return
     * @throws AuthenticationException
     */
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {
        System.out.println("执行认证逻辑");
        String name = "xiaobai";
        String password = "123456";
        //判断密码
        UsernamePasswordToken token = (UsernamePasswordToken) authenticationToken;
        if (!token.getUsername().equals(name)) {
            //shiro底层会抛出UnKnowAccountException
            return null;
        }
        //判断密码
        return new SimpleAuthenticationInfo("",password,"");
    }
```

## 实例2 整合MyBatis plus实现登录

### 1. 导入mybatis相关的依赖

```xml
<!--mysql驱动-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.11</version>
        </dependency>
        <!--jdbc-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jdbc</artifactId>
        </dependency>
        <!--mybatis plus-->
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
            <version>3.3.2</version>
        </dependency>
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus</artifactId>
            <version>3.3.2</version>
        </dependency>
```

### 2.配置application.properties

```properties
#mybatis
#和数据库表对应的pojo对象
mybatis.type-aliases-package=com.ttt.pojo
mybatis.mapper-locations=classpath:com/ttt/mapper/*.xml
#可以自动识别(mybatis自带连接池)
spring.datasource.driver-class-name =com.mysql.cj.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost:3306/studentinfo?useUnicode=true&characterEncoding=utf-8&serverTimezone=UTC
spring.datasource.username =root
spring.datasource.password =123456
##如果不使用默认的数据源 （com.zaxxer.hikari.HikariDataSource）
spring.datasource.type =com.zaxxer.hikari.HikariDataSource
#控制台打印sql
mybatis.configuration.log-impl=org.apache.ibatis.logging.stdout.StdOutImpl
```

### 3.编写User实体

这里使用lombok

```java
/**
 * (Studen)实体类
 *
 * @author ttt
 * @since 2020-07-17 15:55:39
 */
@Data
@AllArgsConstructor
@NoArgsConstructor
public class Studen implements Serializable {
    /**
    * 学号
    */
    private Integer sid;
    /**
    * 学员姓名
    */
    private String sname;
    /**
    * 密码
    */
    private String spas;
    /**
    * 学员性别
    */
    private String sgender;
    /**
    * 学员年龄
    */
    private Integer sage;
}
```

### 4.编辑接口

```java
@Repository
public interface StudentMapper extends BaseMapper<Studen> {
}
```

### 5.编写业务接口和实现

```java
@Service
public class StudentService {
    @Autowired
    private StudentMapper studentMapper;

    public Studen getStudentByName(String name){
        Studen studen = null;
        try {
            studen = studentMapper.selectList(new QueryWrapper<Studen>().eq("sname", name)).get(0);
        } catch (Exception e) {
        }
        return studen;
    }
}
```

### 6.添加@MapperScan注解

```java
package com.ttt;

import org.mybatis.spring.annotation.MapperScan;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@MapperScan("com.ttt.mapper")
@SpringBootApplication
public class ShiroDemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(ShiroDemoApplication.class, args);
    }

}
```

### 7.修改UserRealm

```java
@Autowired
    private StudentService studentService;
    /**
     * 认证逻辑
     * @param authenticationToken
     * @return
     * @throws AuthenticationException
     */
    @SneakyThrows
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {
        System.out.println("执行认证逻辑");
        //判断密码
        UsernamePasswordToken token = (UsernamePasswordToken) authenticationToken;
        Studen studen = studentService.getStudentByName(token.getUsername());
        System.out.println(studen);
        if (studen == null) {
            //shiro底层会抛出UnKnowAccountException
            return null;
        }
        //判断密码
        return new SimpleAuthenticationInfo("",studen.getSpas(),"");
    }
```

## 用户授权

### 1.使用Shiro内置过滤器拦截资源

授权过滤器

注意：当前授权拦截后，shiro会自动跳转到未授权页面

```java
/**
     * 创建ShiroFilterFactoryBean
     * @return 创建
     */
    @Bean
    public ShiroFilterFactoryBean getShiroFilterFactoryBean(@Qualifier("securityManager")DefaultWebSecurityManager securityManager) {
        ShiroFilterFactoryBean shiroFilterFactoryBean = new ShiroFilterFactoryBean();
        //设置安全管理器
        shiroFilterFactoryBean.setSecurityManager(securityManager);
        Map<String, String> filterMap = new LinkedHashMap<String, String>();
        //第一种方法
        /*filterMap.put("/add", "authc");
        filterMap.put("/update", "authc");*/
        //第二种方法
        filterMap.put("/ThymeleafDemo", "anon");
        filterMap.put("/login", "anon");

        //授权过滤器
        filterMap.put("/add", "perms[user:add]");

        filterMap.put("/*", "authc");
        //设置登录方法
        shiroFilterFactoryBean.setLoginUrl("/toLogin");
        //设置未授权页面
        shiroFilterFactoryBean.setUnauthorizedUrl("/noAuth");
        shiroFilterFactoryBean.setFilterChainDefinitionMap(filterMap);
        return shiroFilterFactoryBean;
    }
```

### 2.完成Shiro的资源授权

```java
/**
     * 授权逻辑
     * @param principalCollection
     * @return
     */
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
        System.out.println("执行授权逻辑");
        //给资源进行授权
        SimpleAuthorizationInfo info = new SimpleAuthorizationInfo ();
        info.addStringPermission("user:add");
        return info;
    }
```

