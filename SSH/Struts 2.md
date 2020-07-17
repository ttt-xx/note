# Struts 2笔记

## Struts 2及其优势 

Struts 2是一个MVC框架，以WebWork框架的设计思想为核心，吸收了Struts 1的部分优点
Struts 2拥有更加广阔的前景，自身功能强大

## Struts 2的获取 

Struts 2官方地址：http://struts.apache.org
本书选取Struts 2.3.16.3进行讲解
Struts 2 目录结构

![1592902339546](C:\Users\XXX\AppData\Roaming\Typora\typora-user-images\1592902339546.png)

## 使用Struts 2 开发程序的基本步骤
加载Struts 2 类库
配置web.xml文件
开发视图层页面
开发控制层Action
配置struts.xml文件
部署、运行项目

### 加载Struts2 类库

| **文件名**                     | **说  明**                           |
| ------------------------------ | ------------------------------------ |
| **struts2-core-xxx.jar**       | **Struts 2**框架的核心类库           |
| **xwork-core-xxx.jar**         | **XWork**类库，Struts 2的构建基础    |
| **ognl-xxx.jar**               | **Struts 2**使用的一种表达式语言类库 |
| **freemarker-xxx.jar**         | **Struts 2**的标签模板使用类库       |
| **javassist-xxx.GA.jar**       | **对字节码进行处理**                 |
| **commons-fileupload-xxx.jar** | **文件上传时需要使用**               |
| **commons-io-xxx.jar**         | **Java IO**扩展                      |
| **commons-lang-xxx.jar**       | **包含了一些数据类型的工具类**       |

### 配置web.xml

```xml
<filter>
	<filter-name>struts2</filter-name>
	<filter-class>
		org.apache.struts2.dispatcher.ng.filter.
					StrutsPrepareAndExecuteFilter
	</filter-class>
</filter>
	
<filter-mapping>
	<filter-name>struts2</filter-name>
	<url-pattern>/*</url-pattern>
</filter-mapping>
```

### 编写输出页面helloWorld.jsp

```jsp
…
	<div>
		<h1>
			<!--显示Struts Action中message的属性内容-->
			<s:property value="message"/>
		</h1>
	</div>
	<div>
		<form action="helloWorld.action" method="post">
			请输入您的姓名:
			<input name="name" type="text" />
			<input type="submit" value="提交" />
		</form>
	</div>
…
```

### 编写HelloWorldAction

```java
public class HelloWorldAction implements Action {
	// 用户输入的姓名
	private String name = "";
	// 向用户显示的信息
	private String message = "";
	public String execute() {
		// 根据用户输入的姓名,进行"Hello,XXXX!"的封装
		this.setMessage("Hello,"+this.getName()+"!");
		// 处理完毕,返回导航结果的逻辑名
		return "success";
	}
	…	//省略setter、getter方法
}
```

### 配置Struts 2配置文件（struts.xml）

```java
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE struts PUBLIC
    "-//Apache Software Foundation//DTD Struts Configuration 2.0//EN"
    "http://struts.apache.org/dtds/struts-2.0.dtd">
<struts>
	<package name="default" namespace="/" extends="struts-default">
		<action name="helloWorld" 
					class="cn.strutsdemo.HelloWorldAction">	
			<result name="success">helloWorld.jsp</result>
		</action>
	</package>
</struts>
```

部署、运行项目

### Struts 2开发小结
开发Struts 2应用的基本环节
确认环境
是否添加了Struts 2框架支持文件
是否配置了Filter
编写视图
功能实现
编写Action类
配置struts.xml文件
部署、运行
### 开发控制层Action-LoginAction 

```java
public class LoginAction implements Action {
	 private String username = "";
	 private String password = "";
	 public String execute() {
		if("jason".equals(username) &&"2010".equals(password)) {
			return "success";
		} else {
			return "error";
		}
	}
	……//省略getter/setter方法
} 	
```

### 配置Struts 2配置文件（struts.xml）

```java
<package name="default" namespace="/" extends="struts-default">
	   <action name="login" class="cn.strutsdemo.LoginAction">
		  <!-- 结果为"success"时，跳转至success.jsp页面 -->
		  <result name="success">success.jsp</result>
		  <!-- 结果为"error"时，跳转至fail.jsp页面 -->
		  <result name="error">fail.jsp</result>
	   </action>
</package>
```

## Struts 2访问Servlet API

与Servlet API解耦的访问方式
对Servlet API进行封装
提供了三个Map对象访问request、session、application作用域
通过ActionContext类获取这三个Map对象
Object get("request")
Map getSession()
Map getApplication()

```java
public class LoginAction implements Action {
	private static final String CURRENT_USER = "CURRENT_USER";	
	… //省略username、password属性及其setter和getter方法
	public String execute() {
		if("jason".equals(username) && "2010".equals(password)) {
			Map<String,Object> session = null;
			session = ActionContext.getContext().getSession();
			if(session.containsKey(CURRENT_USER)) {				
				session.remove(CURRENT_USER);
			}
			session.put(CURRENT_USER, username);
			return "success";
		} else {
			return "error";
		}
	}
}
```

页面显示

```html
<body>
	<h1>读取Session中保存的用户名</h1>
	<div>欢迎您，${sessionScope.CURRENT_USER}！</div>
</body>
```

与Servlet API耦合的访问方式
通过ServletActionContext类获取Servlet API对象
 ServletContext getServletContext()
 HttpServletResponse getResponse()
 HttpServletRequest  getRequest()
通过request.getSession()获取session对象
通过xxx.setAttribute()和xxx.getAttribute() 方法在不同的页面或Action中传递数据

```java
public class LoginAction implements Action {
	private static final String CURRENT_USER = "CURRENT_USER";	
	… //省略username、password属性及其setter和getter方法
	public String execute() {			
		if("jason".equals(username) && "2010".equals(password)) {	
			HttpSession session = null;
			session = ServletActionContext.getRequest().getSession(); 			if(session.getAttribute(CURRENT_USER) != null) {
				session.removeAttribute(CURRENT_USER);
			}
			session.setAttribute(CURRENT_USER, username);	
			return "success";
		} else {			
			return "error";
		}
	}
}
```

## Struts 2的数据校验

Struts 2提供了数据验证机制
继承ActionSupport类来完成Action开发
ActionSupport类不仅对Action接口进行简单实现，同时增加了验证、本地化等支持 

## Struts 2标签 

Struts 2标签分为UI标签和通用标签

```jsp
 <%@ taglib prefix="s" uri="/struts-tags"%>
```

常用表单标签

| **标 签** | **说  明**       |
| --------- | ---------------- |
| **…**     | **表单标签**     |
| **…**     | **文本输入框**   |
| **…**     | **密码输入框**   |
| **…**     | **文本域输入框** |
| **…**     | **单选按钮**     |
| **…**     | **多选框**       |
| ****      | **提交标签**     |
| ****      | **重置标签**     |
| ****      | **隐藏域标签**   |

常用非表单标签

| **标 签** | **说  明**                 |
| --------- | -------------------------- |
| ****      | **显示****Action****错误** |
| ****      | **显示****Action****消息** |
| ****      | **显示字段错误**           |

常用通用标签

| **名称**     | **标 签**      | **说  明**                                  |
| ------------ | -------------- | ------------------------------------------- |
| **条件标签** | ******……****** | **根据表达式的值， ****判断将要执行的内容** |
| **迭代**     | **……**         | **用于遍历集合**                            |

## 条件标签

条件判断标签

```jsp
<s:if test="表达式">
   	需要执行的代码
</s:if>
<s:elseif test="表达式">
  		需要执行的代码
</s:elseif>
<s:else>
   	需要执行的代码
</s:else>
```

iterator

```jsp
<s:iterator value="集合对象" status="status" id="name">
   		读取集合对象并输出显示
</s:iterator>
```

value属性：需要进行遍历的集合对象
status属性：当前迭代元素的IteratorStatus实例
id属性:当前迭代元素的id，可直接访问元素，该参数可选

## Struts 2基本架构

![1593479595085](C:\Users\XXX\AppData\Roaming\Typora\typora-user-images\1593479595085.png)

核心控制器
需要在web.xml中进行配置
对框架进行初始化，以及处理所有的请求

```xml
<filter>
	<filter-name>struts2</filter-name>
	<filter-class>
		org.apache.struts2.dispatcher.ng.filter.
					StrutsPrepareAndExecuteFilter
	</filter-class>
</filter>	
<filter-mapping>
	<filter-name>struts2</filter-name>
	<url-pattern>/*</url-pattern>
</filter-mapping>
```

Struts 2.0版本的核心控制器为org.apache.struts2.dispatcher.FilterDispatcher

## Action
创建Action
实现Action接口
继承ActionSupport类
配置Action

```xml
…
<struts>
	<package name="default" namespace="/" extends="struts-default">
		<action name="login" class="cn.houserent.action.LoginAction">
			<result name="success">/page/manage.jsp</result>
			<result name="input">/page/login.jsp</result>						       <result name="error">/page/error.jsp</result>
		</action>
	</package>
</struts>
```

Result
实现对结果的调用
result元素的值指定对应的实际资源位置
name属性表示result逻辑名

```xml
…
<struts>
	<package name="default" namespace="/" extends="struts-default">
		<action name="login" class="cn.houserent.action.LoginAction">
			<result name="success">/page/manage.jsp</result>
			<result name="input">/page/login.jsp</result>								<result name="error">/page/error.jsp</result>
		</action>
	</package>
</struts>
```

struts.xml
核心配置文件，主要负责管理Action
通常放在WEB-INF/classes目录下，在该目录下的struts.xml文件可以被自动加载

```xml
…
<struts>
  	<constant name="" value=""/>
	<package name="" namespace="" extends="">
		<action name="" class="">
			<result name=""></result>			
		</action>
	</package>
</struts>
```

struts.xml
constant元素
配置常量，可以改变Struts 2框架的一些行为
name属性表示常量名称，value属性表示常量值 	

```xml
…
<struts>
  	<constant name="struts.i18n.encoding" value="UTF-8"/> 	<package name="" namespace="" extends="">
		<action name="" class="">
			<result name=""></result>			
		</action>
	</package>
</struts>
```

struts.xml
package元素 
包的作用：简化维护工作，提高重用性
包可以“继承”已定义的包，并可以添加自己包的配置
name属性为必需的且唯一，用于指定包的名称
extends属性指定要扩展的包
namespace属性定义该包中action的命名空间 ，可选

```xml
…
<struts>
  	<constant name="" value=""/>
	<package name="default" namespace="/" extends="struts-default">
		<action name="" class="">
			<result name=""></result>			
		</action>
	</package>
</struts>
```

struts-default.xml 
Struts 2默认配置文件，会自动加载
struts-default包在struts-default.xml文件中定义
struts-plugin.xml 
Struts 2插件使用的配置文件 
加载顺序

struts-default.xml

struts-plugin.xml

struts.xml

### Action的作用
封装工作单元
数据转移的场所 
返回结果字符串 

```java
public class HelloWorldAction implements Action {	
	  private String name = "";
	  private String message = "";	
	  public String execute() {
		  this.setMessage("Hello,"+this.getName()+"！");
		  return SUCCESS;
	  }
	  //...省略setter/getter方法 
}
```

### method属性
实现Action中不同方法的调用
特点
避免动态方法调用的安全隐患
导致大量的Action配置 

```xml
<action name="login" 
	  class="cn.houserent.action.UserAction" method="login">
	……
</action>
<action name="register" 
	class="cn.houserent.action.UserAction" method="register">
	 ……
</action>
```

### 动态方法调用
作用：减少Action数量
使用：actionName!methodName.action 
禁用：将常量struts.enable.DynamicMethodInvocation设置为false 

```java
public class UserAction implements Action {	
	  …	
	public String login() {…  }
	public String register() {…}
}
```

```xml
<action name="user" class="cn.houserent.action.UserAction">
	  <result name="login">/page/manage.jsp</result>
	……
</action>
```

### 通配符(*)
作用：另一种形式的动态方法调用

```xml
<action name= "*User" 
		class="cn.houserent.action.UserAction" method="{1}">
	  <result>/page/{1}_success.jsp</result>
	  <result name="input">/page/{1}.jsp</result>
</action>
```

配置默认Action
没有Action匹配请求时，默认Action将被执行
通过<default-action-ref … />元素配置默认Action

```xml
<struts>
  	<default-action-ref name="defaultAction"/ >
	<package name="default" extends="struts-default">
		<action name="defaultAction">
			<result>error.jsp</result>			
		</action>
	</package>
</struts>
```

## Result常用结果类型

常用结果类型 
dispatcher类型
默认结果类型，后台使用RequestDispatcher转发请求 
redirect类型 
后台使用的sendRedirect()将请求重定向至指定的URL 
redirectAction类型 
主要用于重定向到Action 

```xml
<action name="*User" 
		class="cn.houserent.action.UserAction" method="{1}">
	              <result name="success" type="dispatcher">
				/page/{1}_success.jsp
			</result>
			<result name="input">/page/{1}.jsp</result>
			<result name="error">/page/error.jsp</result>
</action>
```

### 动态结果
配置时不知道执行后的结果是哪一个，运行时才知道哪个结果作为视图显示给用户

```java
public class UserAction extends ActionSupport {
	private String nextDispose;
	public String login() {
		...
		if(user.isManager()){
			nextDispose = "manager";
		}else{
			nextDispose = "common";
		}
		return SUCCESS;
	}
	public String getNextDispose(){
		return nextDispose;
	}
	...
}
```

${nextDispose}表示调用Action中的getNextDispose方法，获取导航信息

### 全局结果 
实现同一个包中多个action共享一个结果

```xml
<struts>	
	<default-action-ref name="defaultAction"/ >
	<package name="default" extends="struts-default">
		<global-results>
			<result name="error">/page/error.jsp</result>
			<result name="login" type="redirect">/page/login.jsp</result>
		</global-results>	
			
		…省略action的配置…
	</package>
</struts>
```

## OGNL简介

### 什么是OGNL 
Object Graph Navigation Language
开源项目，取代页面中Java脚本，简化数据访问
和EL同属于表达式语言，但功能更为强大 

OGNL在Struts 2中做的两件事情
表达式语言
将表单或Struts 2标签与特定的Java数据绑定起来，用来将数据移入、移出框架 
类型转换 
数据进入和流出框架，在页面数据的字符串类型和Java数据类型之间进行转换

### OGNL在框架中的作用

OGNL融入Struts 2
数据流入
数据流出 

![1593592985281](C:\Users\XXX\AppData\Roaming\Typora\typora-user-images\1593592985281.png)

### 值栈与OGNL

#### 值栈(ValueStack)
由Struts 2框架创建的存储区域，具有栈的特点
Action的实例会被存放到值栈中

#### OGNL访问值栈
按照从上到下的顺序
靠近栈顶的同名属性会被读取

![1593593006282](C:\Users\XXX\AppData\Roaming\Typora\typora-user-images\1593593006282.png)

### 为什么进行类型转换 

在基于HTTP协议的Web应用中
客户端请求的所有内容都以文本编码方式传输到服务器端
服务器端的编程语言却有着丰富的数据类型

Servlet中，类型转换工作由开发者自己完成

```java
String agestr = request.getParameter("age");
int age = Integer.parseInt(agestr);
```

### 内置类型转换器

Struts 2提供了多种内置类型转换器，自动对客户端传来的数据进行类型转换

| **内置类型转换器**                                     | **说  明**                                                   |
| ------------------------------------------------------ | ------------------------------------------------------------ |
| **String**                                             | **将**int、long、double、boolean、String类型的数组或者java.util.Date类型转换为字符串 |
| **boolean**/Boolean                                    | **在字符串和布尔值之间进行转换**                             |
| **char/Character**                                     | **在字符串和字符之间进行转换**                               |
| **int/Integer**、float/Float、long/Long、double/Double | **在字符串和数值型数据之间进行转换**                         |
| **Date**                                               | **在字符串和日期类型之间进行转换。具体输入输出格式与当前的**Locale相关 |
| **数组和集合**                                         | **在字符串数组和数组对象、集合对象间进行转换**               |

