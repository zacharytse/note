[TOC]
# ModelAndView
spring提供的model对象
```java{.line-numbers}
@RequestMapping("/modelAndView")
    public ModelAndView testModelAndView(){
        ModelAndView mv = new ModelAndView();
        mv.addObject("username","张三");
        mv.setViewName("success");
        return mv;
    }
```
```html{.line-numbers}
<%--
  Created by IntelliJ IDEA.
  User: xie
  Date: 2020/11/2
  Time: 19:47
  To change this template use File | Settings | File Templates.
--%>
<%@ page contentType="text/html;charset=UTF-8" language="java" isELIgnored="false" %>
<html>
<head>
    <meta http-equiv="Content-Type" content="text/html;charset=UTF-8">
    <title>Title</title>
</head>
<body>
    <h3>success 了</h3>

    ${username}
</body>
</html>
```
这里要注意设置isELIgnored="false" 
# 使用ajax对异步数据进行处理
```java{.line-numbers}
package com.xcq.controller;

import com.xcq.domain.User;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
@RequestMapping("/user")
public class UserController {

    @RequestMapping("/testAjax")
    public @ResponseBody User testAjax(@RequestBody User user){
        user.setUsername("dsa");
        user.setAge(20);
        return user;
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

    <!--定义不拦截的静态资源-->
    <mvc:resources mapping="/js/**" location="/js/"></mvc:resources>

    <!--开启springmvx注解的支持-->
    <mvc:annotation-driven>
    </mvc:annotation-driven>
</beans>
```
 <mvc:resources mapping="/js/**" location="/js/"></mvc:resources>防止jquery的静态资源被前端控制器拦截
```html{.line-numbers}
<%--
  Created by IntelliJ IDEA.
  User: xie
  Date: 2020/11/5
  Time: 21:34
  To change this template use File | Settings | File Templates.
--%>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
    <script src="js/jquery.min.js"></script>
    <script>
        $(function () {
            $("#btn").click(function () {
                alert("点击了按钮")
                $.ajax({
                    url:"user/testAjax",
                    contentType:"application/json;charset=utf-8",
                    data:'{"username":"张三","password":"1234","age":30}',
                    dataType:"json",
                    type:"post",
                    success:function (data) {
                        alert(data.username)
                        alert(data.password)
                        alert(data.age)
                    }
                });
            });
        });
    </script>
</head>
<body>
    <%--<a href="user/testAjax">test ajax</a>--%>
    <button id="btn">测试ajax</button>
</body>
</html>
```