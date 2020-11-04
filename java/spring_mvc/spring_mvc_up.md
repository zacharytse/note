[TOC]
# 三层架构
![](https://gitee.com/zacharytse/image/raw/master/img/fgsdfgfsg.jpg)
# 是什么
springmvc是一种基于java实现的mvc的轻量级框架
# springmvc详细的过程
![](https://gitee.com/zacharytse/image/raw/master/img/springmvc执行流程原理.jpg)
```java{.line-numbers}
package com.xcq.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;

/**
 * 控制器
 */
@Controller("helloController")
public class HelloController {

    @RequestMapping(path = "/hello")
    public String sayHello(){
        System.out.println("Hello world");
        return "success";
    }
}
```
```xml{.line-numbers}
<!--springmvc.xml-->
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/mvc
       http://www.springframework.org/schema/mvc/spring-mvc.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd">

    <!--开启注解扫描-->
    <context:component-scan base-package="com.xcq"></context:component-scan>

    <!--视图解析器-->
    <bean id="internalResourceViewResolver"
          class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/pages/"></property>
        <property name="suffix" value=".jsp"></property>
    </bean>

    <!--开启springmvx注解的支持-->
    <mvc:annotation-driven></mvc:annotation-driven>
</beans>
```
```xml{.line-numbers}
<!--web.xml-->
<!DOCTYPE web-app PUBLIC
 "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
 "http://java.sun.com/dtd/web-app_2_3.dtd" >

<web-app>
  <display-name>Archetype Created Web Application</display-name>
  <servlet>
    <servlet-name>dispatcherServlet</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <!--通过DispatcherServlet加载springmvc.xml的配置文件-->
    <init-param>
      <param-name>contextConfigLocation</param-name>
      <param-value>classpath:springmvc.xml</param-value>
    </init-param>
    <!--启动服务器时servlet就会被创建为对象-->
    <load-on-startup>1</load-on-startup>
  </servlet>
  <servlet-mapping>
    <servlet-name>dispatcherServlet</servlet-name>
    <url-pattern>/</url-pattern>
  </servlet-mapping>
</web-app>
```
```html{.line-numbers}
<!--index.jsp-->
<%--
  Created by IntelliJ IDEA.
  User: xie
  Date: 2020/11/2
  Time: 19:33
  To change this template use File | Settings | File Templates.
--%>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
    <h3>入门</h3>

    <a href="hello">test</a>
</body>
</html>
```
```html{.line-numbers}
<!--success.jsp-->
<%--
  Created by IntelliJ IDEA.
  User: xie
  Date: 2020/11/2
  Time: 19:47
  To change this template use File | Settings | File Templates.
--%>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
    <h3>success 了</h3>
</body>
</html>
```
# 请求参数的绑定
```html{.line-numbers}
<!--index.jsp-->
<%--
  Created by IntelliJ IDEA.
  User: xie
  Date: 2020/11/2
  Time: 19:33
  To change this template use File | Settings | File Templates.
--%>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
   <h3>Demo</h3>
   <%--<a href="param/testParams?username=root&password=1234">测试参数</a>
   --%>
    <form action="form/testForm" method="post">
        姓名:<input type="text" name="username"><br>
        用户名: <input type="text" name="password"><br>
        金额:<input type="text" name="money"><br>
        账户名:<input type="text" name="user.uname"><br>
        <input type="submit"><br>
    </form>
</body>
</html>

```
```xml{.line-numbers}
<!--web.xml-->
<!DOCTYPE web-app PUBLIC
 "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
 "http://java.sun.com/dtd/web-app_2_3.dtd" >

<web-app>
  <display-name>Archetype Created Web Application</display-name>
  <filter>
    <filter-name>characterEncodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
      <param-name>encoding</param-name>
      <param-value>UTF-8</param-value>
    </init-param>
  </filter>
  <filter-mapping>
    <filter-name>characterEncodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
  </filter-mapping>
  <servlet>
    <servlet-name>dispatcherServlet</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <!--通过DispatcherServlet加载springmvc.xml的配置文件-->
    <init-param>
      <param-name>contextConfigLocation</param-name>
      <param-value>classpath:springmvc.xml</param-value>
    </init-param>
    <!--启动服务器时servlet就会被创建为对象-->
    <load-on-startup>1</load-on-startup>
  </servlet>
  <servlet-mapping>
    <servlet-name>dispatcherServlet</servlet-name>
    <url-pattern>/</url-pattern>
  </servlet-mapping>

</web-app>
```
```java{.line-numbers}
package com.xcq.controller;

import com.xcq.domain.Account;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller
@RequestMapping("/form")
public class FormController {

    @RequestMapping("/testForm")
    public String testForm(Account account){
        System.out.println(account);
        return "success";
    }
}
```
```java{.line-numbers}
package com.xcq.domain;

import java.io.Serializable;

public class Account implements Serializable {
    private String username;
    private String password;
    private Float money;
    private User user;

    public User getUser() {
        return user;
    }

    public void setUser(User user) {
        this.user = user;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    public Float getMoney() {
        return money;
    }

    public void setMoney(Float money) {
        this.money = money;
    }

    @Override
    public String toString() {
        return "Account{" +
                "username='" + username + '\'' +
                ", password='" + password + '\'' +
                ", money=" + money +
                ", user=" + user +
                '}';
    }
}
```
```java{.line-numbers}
package com.xcq.domain;

import java.io.Serializable;

public class User implements Serializable {
    private String uname;

    public String getUname() {
        return uname;
    }

    public void setUname(String uname) {
        this.uname = uname;
    }

    @Override
    public String toString() {
        return "User{" +
                "uname='" + uname + '\'' +
                '}';
    }
}                   
``` 
# 定义转换器
```java{.line-numbers}
package com.xcq.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;

import java.util.Date;

@Controller("converterController")
@RequestMapping("/converter")
public class ConverterController {

    @RequestMapping("/testConverter")
    public String testConverter(Date date){
        System.out.println(date);
        return "success";
    }
}
```
```java{.line-numbers}
package com.xcq.converter;


import org.springframework.core.convert.converter.Converter;
import org.springframework.util.StringUtils;

import java.text.DateFormat;
import java.text.SimpleDateFormat;
import java.util.Date;

public class StringToDateConverter implements Converter<String, Date> {
    @Override
    public Date convert(String s) {
        DateFormat format = null;
        try{
            if(StringUtils.isEmpty(s)){
                throw new RuntimeException("please input date");
            }
            format = new SimpleDateFormat("yyyy-MM-dd");
            Date date = format.parse(s);
            return date;
        }catch (Exception e){
            throw new RuntimeException("date is error");
        }
    }
}
```
```xml{.line-numbers}
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/mvc
       http://www.springframework.org/schema/mvc/spring-mvc.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd">

    <!--开启注解扫描-->
    <context:component-scan base-package="com.xcq"></context:component-scan>

    <!--视图解析器-->
    <bean id="internalResourceViewResolver"
          class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/pages/"></property>
        <property name="suffix" value=".jsp"></property>
    </bean>

    <bean id="conversionService"
          class="org.springframework.context.support.ConversionServiceFactoryBean">
        <!--注入一个类型转换器-->
        <property name="converters">
            <bean class="com.xcq.converter.StringToDateConverter"></bean>
        </property>
    </bean>
    <!--开启springmvx注解的支持-->
    <mvc:annotation-driven conversion-service="conversionService">
    </mvc:annotation-driven>
</beans>
```
```html{.line-numbers}
<%--
  Created by IntelliJ IDEA.
  User: xie
  Date: 2020/11/2
  Time: 19:33
  To change this template use File | Settings | File Templates.
--%>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
   <h3>Demo</h3>
   <%--<a href="param/testParams?username=root&password=1234">测试参数</a>
   --%>
    <%--<form action="form/testForm" method="post">
        姓名:<input type="text" name="username"><br>
        用户名: <input type="text" name="password"><br>
        金额:<input type="text" name="money"><br>
        账户名:<input type="text" name="user.uname"><br>
        <input type="submit"><br>
    </form>--%>
    <%--<a href="method/testMethod?accountName=test">测试</a>--%>
   <form action="converter/testConverter" method="post">
       姓名:<input type="text" name="username"><br>
       用户名: <input type="text" name="password"><br>
       金额:<input type="text" name="money"><br>
       日期:<input type="text" name="date"><br>
       <input type="submit"><br>
   </form>
</body>
</html>
```