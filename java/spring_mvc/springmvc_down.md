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
# 传统方法的文件上传
```html{.line-numbers}
<%--
  Created by IntelliJ IDEA.
  User: xie
  Date: 2020/11/6
  Time: 9:57
  To change this template use File | Settings | File Templates.
--%>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
    <form action="user/upload" enctype="multipart/form-data" method="post">
        选择文件:<input name="upload" type="file"><br>
        <input type="submit" value="上传文件">
    </form>
</body>
</html>
```
表单的enctype要设置成multipart/form-data，且method要设置为post
```java{.line-numbers}
package com.xcq.controller;

import org.apache.commons.fileupload.FileItem;
import org.apache.commons.fileupload.FileUploadException;
import org.apache.commons.fileupload.disk.DiskFileItemFactory;
import org.apache.commons.fileupload.servlet.ServletFileUpload;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

import javax.servlet.http.HttpServletRequest;
import java.io.File;
import java.util.List;
import java.util.UUID;

@Controller
@RequestMapping("/user")
public class UploadController {

    @RequestMapping("/upload")
    public String testUpload(HttpServletRequest request) throws Exception {
        //先获取到要上传的文件目录
        String path = request.getSession().
                getServletContext().getRealPath("/uploads");
        //创建file对象
        File file = new File(path);
        if(!file.exists()){
            file.mkdirs();
        }
        //创建磁盘文件项工厂
        DiskFileItemFactory factory = new DiskFileItemFactory();
        ServletFileUpload fileUpload = new ServletFileUpload(factory);
        //解析request对象
        List<FileItem> list = fileUpload.parseRequest(request);
        //遍历
        for(FileItem item : list){
            if(!item.isFormField()){
                //上传文件项
                String fileName = item.getName();
                String uuid = UUID.randomUUID().
                        toString().replace("-","");
                item.write(new File(file,uuid + "_" + fileName));
                //删除临时文件
                item.delete();
            }
        }
        return "success";
    }
}
```
# springmvc的方式上传
jsp和传统方法一致
配置文件需要加上配置文件解析器
```xml{.line-numbers}
<bean id="multipartResolver"
          class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
        <property name="maxUploadSize" value="10485760"/>
</bean>
```
配置文件解析器的id必须为multipartResolver

Controller类的编写如下
```java{.line-numbers}
@RequestMapping("/upload2")
    public String testUpload2(HttpServletRequest request, MultipartFile upload) throws IOException {
        String path = request.getSession().
                getServletContext().getRealPath("/uploads");
        File file = new File(path);
        if (!file.exists()) {
            file.mkdirs();
        }
        String filename = upload.getName();
        String uuid = UUID.randomUUID().toString().replace("-", "");
        upload.transferTo(new File(file, uuid + "_" + filename));
        return "success";
    }
```
# 跨服务器上传文件
```java{.line-numbers}
@RequestMapping("/upload3")
    public String testUpload3(MultipartFile upload) throws IOException {
        //定义图片服务器的请求路径
        String path = "http://localhost:9090/springmvc_image/uploads/";

        String filename = upload.getOriginalFilename();
        String uuid = UUID.randomUUID().toString().replace("-", "");
        filename = uuid + "_" + filename;
        //创建客户端对象
        Client client = Client.create();
        //连接图片服务器
        WebResource webResource = client.resource(path + filename);
        //上传文件
        webResource.put(upload.getBytes());
        return "success";
    }
```
# 异常处理器
一般持久层，业务层，表现层的异常都应该由异常处理器来处理，并且不应该直接返回到浏览器端
![](https://gitee.com/zacharytse/image/raw/master/img/20201106142749.png)
1. 编写自定义的异常类
```java{.line-numbers}
package com.xcq.exception;

public class SysException extends Exception{

    //异常提示信息
    private String message;

    @Override
    public String getMessage() {
        return message;
    }

    public void setMessage(String message) {
        this.message = message;
    }

    public SysException(String message) {
        this.message = message;
    }
}
```
2. 编写异常处理类，要实现HandlerExceptionResolver接口
```java{.line-numbers}
package com.xcq.exception;

import org.springframework.web.servlet.HandlerExceptionResolver;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

public class SysExceptionResolver implements HandlerExceptionResolver {

    @Override
    public ModelAndView resolveException(HttpServletRequest httpServletRequest,
                                         HttpServletResponse httpServletResponse,
                                         Object o, Exception e) {
        e.printStackTrace();
        SysException ex = null;
        if(e instanceof SysException){
            ex = (SysException)e;
        } else {
            ex = new SysException("404");
        }
        ModelAndView mv = new ModelAndView();
        //存入错误的提示信息
        mv.addObject("message",ex.getMessage());
        mv.setViewName("error");
        return mv;
    }
}
```
在xml文件中配置异常处理器
```xml
<bean id="sysExceptionResolver"
          class="com.xcq.exception.SysExceptionResolver"/>
```
测试Controller类
```java{.line-numbers}
package com.xcq.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller
@RequestMapping("/user")
public class UserController {

    @RequestMapping("/testExecption")
    public String testExecption(){
        int i = 1 / 0;
        return "success";
    }
}
```
# 拦截器
自定义拦截器类
```java{.line-numbers}
package com.xcq.interceptor;

import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

public class MyInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("preHandle...");
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        System.out.println("postHandle...");
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        System.out.println("afterCompletion...");
    }
}
```
配置拦截器
```xml{.line-numbers}
<mvc:interceptors>
        <mvc:interceptor>
            <mvc:mapping path="/user/*"/>
            <bean class="com.xcq.interceptor.MyInterceptor"/>
        </mvc:interceptor>
</mvc:interceptors>
```

> - preHandle方法时controller方法执行前拦截的方法。可以使用request或者response跳转到指定的页面。return true是放行执行下一个拦截器，如果没有下一个拦截器，则执行controller方法
> - postHandle是controller方法执行后执行的方法，但在jsp视图执行前执行。可以使用request或者response跳转到指定的页面。
> - afterCompletion是在jsp执行后执行。不能使用request或者response跳转。一般用作资源释放





