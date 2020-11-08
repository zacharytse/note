[TOC]
# 简单的入门案例
maven配置文件如下
```xml{.line-numbers}
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.xcq</groupId>
    <artifactId>springboot</artifactId>
    <version>1.0-SNAPSHOT</version>


    <!--从springboot继承默认配置-->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <version>2.3.4.RELEASE</version>
        <artifactId>spring-boot-starter-parent</artifactId>
        <relativePath/><!--从仓库中搜索parent-->
    </parent>

    <dependencies>
        <!--实现对springmvc的自动配置-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>
</project>
```
springboot的基本配置文件application.yaml如下
```yaml{.line-numbers}
server:
  port:8080 #内嵌的Tomcat端口号
```
Controller类如下
```java{.line-numbers}
package com.xcq.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/demo")
public class DemoController {

    @GetMapping("/echo")
    public String echo(){
        return "echo";
    }
}
```
编写Application运行springboot
```java{.line-numbers}
package com.xcq;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class,args);
    }
}
```
# 执行自动化测试
```java{.line-numbers}
package com.xcq.demo.controller;

import com.xcq.demo.DemoApplication;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.ResultActions;
import org.springframework.test.web.servlet.request.MockMvcRequestBuilders;
import org.springframework.test.web.servlet.result.MockMvcResultMatchers;

@RunWith(SpringRunner.class)
@SpringBootTest(classes = DemoApplication.class)
@AutoConfigureMockMvc
public class UserControllerTest {

    @Autowired
    private MockMvc mvc;

    @Test
    public void testList() throws Exception {
        // 查询用户列表
        ResultActions resultActions = mvc.perform(MockMvcRequestBuilders.get("/users"));
        // 校验结果
        resultActions.andExpect(MockMvcResultMatchers.status().isOk()); // 响应状态码 200
        resultActions.andExpect(MockMvcResultMatchers.content().json("[\n" +
                "    {\n" +
                "        \"id\": 1,\n" +
                "        \"username\": \"yudaoyuanma\"\n" +
                "    },\n" +
                "    {\n" +
                "        \"id\": 2,\n" +
                "        \"username\": \"woshiyutou\"\n" +
                "    },\n" +
                "    {\n" +
                "        \"id\": 3,\n" +
                "        \"username\": \"chifanshuijiao\"\n" +
                "    }\n" +
                "]")); // 响应结果
    }

    @Test
    public void testGet() throws Exception {
        // 获得指定用户编号的用户
        ResultActions resultActions = mvc.perform(MockMvcRequestBuilders.get("/users/1"));
        // 校验结果
        resultActions.andExpect(MockMvcResultMatchers.status().isOk()); // 响应状态码 200
        resultActions.andExpect(MockMvcResultMatchers.content().json("{\n" +
                "\"id\": 1,\n" +
                "\"username\": \"username:1\"\n" +
                "}")); // 响应结果
    }

    @Test
    public void testAdd() throws Exception {
        // 获得指定用户编号的用户
        ResultActions resultActions = mvc.perform(MockMvcRequestBuilders.post("/users")
                .param("username", "yudaoyuanma")
                .param("passowrd", "nicai"));
        // 校验结果
        resultActions.andExpect(MockMvcResultMatchers.status().isOk()); // 响应状态码 200
        resultActions.andExpect(MockMvcResultMatchers.content().string("1")); // 响应结果
    }

    @Test
    public void testUpdate() throws Exception {
        // 获得指定用户编号的用户
        ResultActions resultActions = mvc.perform(MockMvcRequestBuilders.put("/users/1")
                .param("username", "yudaoyuanma")
                .param("passowrd", "nicai"));
        // 校验结果
        resultActions.andExpect(MockMvcResultMatchers.status().isOk()); // 响应状态码 200
        resultActions.andExpect(MockMvcResultMatchers.content().string("true")); // 响应结果
    }

    @Test
    public void testDelete() throws Exception {
        // 获得指定用户编号的用户
        ResultActions resultActions = mvc.perform(MockMvcRequestBuilders.delete("/users/1"));
        // 校验结果
        resultActions.andExpect(MockMvcResultMatchers.status().isOk()); // 响应状态码 200
        resultActions.andExpect(MockMvcResultMatchers.content().string("false")); // 响应结果
    }
}
```
# 执行单元测试
```java{.line-numbers}
package com.xcq.demo.controller;

import com.xcq.demo.domain.UserVO;
import com.xcq.demo.service.UserService;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.mockito.Mockito;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.boot.test.mock.mockito.MockBean;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.ResultActions;
import org.springframework.test.web.servlet.request.MockMvcRequestBuilders;
import org.springframework.test.web.servlet.result.MockMvcResultMatchers;

@RunWith(SpringRunner.class)
@WebMvcTest(UserController.class)
public class UserControllerTest2 {

    @Autowired
    private MockMvc mvc;

    @MockBean
    private UserService userService;

    @Test
    public void testGet2() throws Exception {
        System.out.println("before mock:" + userService.get(1));
        Mockito.when(userService.get(1)).thenReturn(
                new UserVO().setId(1).setUsername("username:1"));
        System.out.println("after mock:" + userService.get(1));

        //查询用户列表
        ResultActions resultActions = mvc.
                perform(MockMvcRequestBuilders.get("/users/v2/1"));
        resultActions.andExpect(MockMvcResultMatchers.status().isOk()); // 响应状态码 200
        resultActions.andExpect(MockMvcResultMatchers.content().json("{\n" +
                "    \"id\": 1,\n" +
                "    \"username\": \"username:1\"\n" +
                "}")); // 响应结果
    }
}
```
# 全局统一返回
- 成功时，返回成功的状态码+数据
- 失败时，返回失败的状态码+错误提示
返回的json数据如下:
```json
// 成功响应
{
    code: 0,
    data: {
        id: 1,
        username: "yudaoyuanma"
    }
}

// 失败响应
{
    code: 233666,
    mes
}
```
定义用于保存全局返回结果的类CommonResult
```java{.line-numbers}
package com.xcq.demo.core.vo;

import com.fasterxml.jackson.annotation.JsonIgnore;
import org.springframework.util.Assert;

import java.io.Serializable;

public class CommonResult<T> implements Serializable {
    public static Integer CODE_SUCCESS = 0;

    /**
     * 错误码
     */
    private Integer code;
    /**
     * 错误提示
     */
    private String message;
    /**
     * 返回数据
     */
    private T data;

    /**
     * 将传入的 result 对象，转换成另外一个泛型结果的对象
     *
     * 因为 A 方法返回的 CommonResult 对象，不满足调用其的 B 方法的返回，所以需要进行转换。
     *
     * @param result 传入的 result 对象
     * @param <T> 返回的泛型
     * @return 新的 CommonResult 对象
     */
    public static <T> CommonResult<T> error(CommonResult<?> result) {
        return error(result.getCode(), result.getMessage());
    }

    public static <T> CommonResult<T> error(Integer code, String message) {
        Assert.isTrue(!CODE_SUCCESS.equals(code), "code 必须是错误的！");
        CommonResult<T> result = new CommonResult<>();
        result.code = code;
        result.message = message;
        return result;
    }

    public static <T> CommonResult<T> success(T data) {
        CommonResult<T> result = new CommonResult<>();
        result.code = CODE_SUCCESS;
        result.data = data;
        result.message = "";
        return result;
    }

    @JsonIgnore // 忽略，避免 jackson 序列化给前端
    public boolean isSuccess() { // 方便判断是否成功
        return CODE_SUCCESS.equals(code);
    }

    @JsonIgnore // 忽略，避免 jackson 序列化给前端
    public boolean isError() { // 方便判断是否失败
        return !isSuccess();
    }

    public Integer getCode() {
        return code;
    }

    public void setCode(Integer code) {
        this.code = code;
    }

    public String getMessage() {
        return message;
    }

    public void setMessage(String message) {
        this.message = message;
    }

    public T getData() {
        return data;
    }

    public void setData(T data) {
        this.data = data;
    }

    @Override
    public String toString() {
        return "CommonResult{" +
                "code=" + code +
                ", message='" + message + '\'' +
                ", data=" + data +
                '}';
    }
}
```
定义处理全局返回结果的类GlobalResponseBodyHandler
```java{.line-numbers}
package com.xcq.demo.core.web;

import com.xcq.demo.core.vo.CommonResult;
import org.springframework.core.MethodParameter;
import org.springframework.http.MediaType;
import org.springframework.http.server.ServerHttpRequest;
import org.springframework.http.server.ServerHttpResponse;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.servlet.mvc.method.annotation.ResponseBodyAdvice;

/**
 * 全局统一返回的处理器
 */
@ControllerAdvice(basePackages = "com.xcq.demo.controller")
public class GlobalResponseHandler implements ResponseBodyAdvice {

    /**
     * 返回值为true,表示拦截Controller所有API接口的返回结果
     * @param methodParameter
     * @param aClass
     * @return
     */
    @Override
    public boolean supports(MethodParameter methodParameter, Class aClass) {
        return true;
    }

    /**
     * 这里约定是成功才返回
     * @param o
     * @param methodParameter
     * @param mediaType
     * @param aClass
     * @param serverHttpRequest
     * @param serverHttpResponse
     * @return
     */
    @Override
    public Object beforeBodyWrite(Object o, MethodParameter methodParameter, MediaType mediaType, Class aClass, ServerHttpRequest serverHttpRequest, ServerHttpResponse serverHttpResponse) {
        //如果已经是CommonResult类型，则直接返回
        if(o instanceof CommonResult){
            return o;
        }
        //如果不是，则包装成CommonResult类型
        return CommonResult.success(o);
    }
}
```
定义异常的规则如下
```
/**
 * 服务异常
 *
 * 参考 https://www.kancloud.cn/onebase/ob/484204 文章
 *
 * 一共 10 位，分成四段
 *
 * 第一段，1 位，类型
 *      1 - 业务级别异常
 *      2 - 系统级别异常
 * 第二段，3 位，系统类型
 *      001 - 用户系统
 *      002 - 商品系统
 *      003 - 订单系统
 *      004 - 支付系统
 *      005 - 优惠劵系统
 *      ... - ...
 * 第三段，3 位，模块
 *      不限制规则。
 *      一般建议，每个系统里面，可能有多个模块，可以再去做分段。以用户系统为例子：
 *          001 - OAuth2 模块
 *          002 - User 模块
 *          003 - MobileCode 模块
 * 第四段，3 位，错误码
 *       不限制规则。
 *       一般建议，每个模块自增。
 */
 ```
#  定义全局异常处理
枚举项目中的错误码
```java{.line-numbers}
 package com.xcq.demo.constants;

/**
 * 用于枚举项目中的错误码
 */
public enum  ServiceExceptionEnum {


   // ==========;系统级别 ==========
    SUCCESS(0, "成功"),
    SYS_ERROR(2001001000, "服务端发生异常"),
    MISSING_REQUEST_PARAM_ERROR(2001001001, "参数缺失"),

    // ========== 用户模块 ==========
    USER_NOT_FOUND(1001002000, "用户不存在"),

    // ========== 订单模块 ==========

    // ========== 商品模块 ==========
    ;

    /**
     * 错误码
     */
    private int code;
    /**
     * 错误提示
     */
    private String message;

    ServiceExceptionEnum(int code, String message) {
        this.code = code;
        this.message = message;
    }

    public int getCode() {
        return code;
    }

    public String getMessage() {
        return message;
    }
}
```
自定义异常类
```java{.line-numbers}
package com.xcq.demo.core.exeception;

import com.xcq.demo.constants.ServiceExceptionEnum;

/**
 * 用于定义业务异常
 */
public class ServiceException extends RuntimeException {

    /**
     * 错误码
     */
    private final Integer code;

    public ServiceException(ServiceExceptionEnum serviceExceptionEnum){
        super(serviceExceptionEnum.getMessage());
        //设置错误码
        this.code = serviceExceptionEnum.getCode();
    }

    public Integer getCode() {
        return code;
    }
}
```
全局异常处理类
```java{.line-numbers}
package com.xcq.demo.core.web;

import com.xcq.demo.constants.ServiceExceptionEnum;
import com.xcq.demo.core.exeception.ServiceException;
import com.xcq.demo.core.vo.CommonResult;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.web.bind.MissingServletRequestParameterException;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.ResponseBody;

import javax.servlet.http.HttpServletRequest;

@ControllerAdvice(basePackages = "com.xcq.demo.controller")
public class GlobalExceptionHandler {

    private Logger logger = LoggerFactory.getLogger(getClass());

    /**
     * 处理业务中出现的异常
     *
     * @param req
     * @param ex
     * @return
     */
    @ResponseBody
    @ExceptionHandler(value = ServiceException.class)
    public CommonResult serviceExceptionHandler(HttpServletRequest req, ServiceException ex) {
        logger.debug("[serviceExceptionHandler]", ex);
        //包装CommonResult结果
        return CommonResult.error(ex.getCode(), ex.getMessage());
    }

    /**
     * 处理springmvc参数不正确的异常
     *
     * @param req
     * @param ex
     * @return
     */
    @ResponseBody
    @ExceptionHandler(value = MissingServletRequestParameterException.class)
    public CommonResult missingServletRequestParameterExceptionHandler(HttpServletRequest req, MissingServletRequestParameterException ex) {
        logger.debug("[missingServletRequestParameterExceptionHandler]", ex);
        return CommonResult.error(ServiceExceptionEnum.MISSING_REQUEST_PARAM_ERROR.getCode(),
                ServiceExceptionEnum.MISSING_REQUEST_PARAM_ERROR.getMessage());
    }

    @ResponseBody
    @ExceptionHandler(value = Exception.class)
    public CommonResult exceptionHandler(HttpServletRequest req, Exception e) {
        logger.debug("[exceptionHandler]",e);
        return CommonResult.error(ServiceExceptionEnum.SYS_ERROR.getCode(),
                ServiceExceptionEnum.SYS_ERROR.getMessage());
    }


}
```
# 拦截器
```java{.line-numbers}
// HandlerInterceptor.java

public interface HandlerInterceptor {

	default boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
			throws Exception {
		return true;
	}

	default void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler,
			@Nullable ModelAndView modelAndView) throws Exception {
	}

	default void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler,
			@Nullable Exception ex) throws Exception {
	}

}
```
可以通过一段伪代码看看拦截器的工作
```java{.line-numbers}
// 伪代码
Exception ex = null;
try {
    // 前置处理
    if (!preHandle(request, response, handler)) {
        return;
    }

    // 执行处理器，即执行 API 的逻辑
    handler.execute();

    // 后置处理
    postHandle(request, response, handler);
} catch(Exception exception) {
    // 如果发生了异常，记录到 ex 中
    ex = exception;
} finally {
    afterCompletion(request, response, handler);
}
```
多个HandlerInterceptor可以组成一个拦截器链，整个的执行过程如下:
1. 按照HandlerInterceptor链的正序，执行preHandle()方法
2. 执行handle的逻辑处理
3. 按照HandlerInterceptor链的倒序，执行postHandle()方法
4. 按照HandlerInterceptor链的倒序，执行afterCompletion()方法
   
**代码验证**
先定义3个HandleInterceptor
```java{.line-numbers}
package com.xcq.demo.core.interceptor;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

public class FirstInterceptor implements HandlerInterceptor {

    private Logger logger = LoggerFactory.getLogger(getClass());

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        logger.info("[preHandle][handler({})]",handler);
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        logger.info("[postHandler][handler({})]",handler);
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        logger.info("[afterCompletion][handler({})]",handler,ex);
    }
}
```
```java{.line-numbers}
package com.xcq.demo.core.interceptor;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

public class SecondInterceptor implements HandlerInterceptor {

    private Logger logger = LoggerFactory.getLogger(getClass());

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        logger.info("[preHandle][handler({})]",handler);
        return false;
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        logger.info("[postHandler][handler({})]",handler);
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        logger.info("[afterCompletion][handler({})]",handler,ex);
    }
}
```
```java{.line-numbers}
package com.xcq.demo.core.interceptor;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

public class ThirdInterceptor implements HandlerInterceptor {

    private Logger logger = LoggerFactory.getLogger(getClass());

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        logger.info("[preHandle][handler({})]", handler);
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        logger.info("[postHandle][handler({})]", handler);
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        logger.info("[afterCompletion][handler({})]", handler, ex);
        throw new RuntimeException("故意抛个错误"); // 故意抛出异常
    }
}
```
定义springmvc的配置中心
```java{.line-numbers}
package com.xcq.demo.config;

import com.xcq.demo.core.interceptor.FirstInterceptor;
import com.xcq.demo.core.interceptor.SecondInterceptor;
import com.xcq.demo.core.interceptor.ThirdInterceptor;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration
public class SpringMVCConfiguration implements WebMvcConfigurer {

    @Bean
    public FirstInterceptor firstInterceptor(){
        return new FirstInterceptor();
    }

    @Bean
    public SecondInterceptor secondInterceptor(){
        return new SecondInterceptor();
    }

    @Bean
    public ThirdInterceptor thirdInterceptor(){
        return new ThirdInterceptor();
    }

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        //拦截器1
        registry.addInterceptor(this.firstInterceptor()).addPathPatterns("/**");
        //拦截器2
        registry.addInterceptor(this.secondInterceptor()).addPathPatterns("/users/current_user");
        //拦截器3
        registry.addInterceptor(this.thirdInterceptor()).addPathPatterns("/**");
    }
}
```
先定义一个不使用第二个拦截器的测试接口
```java{.line-numbers}
/**
     * 测试拦截器的接口
     */
    @GetMapping("/do_something")
    public void doSomething(){
        logger.info("[doSomething]");
    }
```
测试结果如下
![](https://gitee.com/zacharytse/image/raw/master/img/20201108184620.png)
再定义一个使用第二个拦截器的接口
```java{.line-numbers}
@GetMapping("/current_user")
    public UserVO currentUser(){
        logger.info("[currentUser]");
        return new UserVO().setId(10).setUsername(UUID.randomUUID().toString());
    }
```
![](https://gitee.com/zacharytse/image/raw/master/img/20201108185126.png)
运行后，可以发现因为只有第一个拦截器成功执行了preHandler()方法，所以也只有第一个拦截器会执行afterCompletion()方法。因为第二个拦截器的preHandler()方法返回false,所以handler()方法根本不会执行，同时第三个拦截器也不会执行。

再定义一个自身会抛出异常的测试接口
```java{.line-numbers}
@GetMapping("/exception-03")
    public void exception03() {
        logger.info("[exception03]");
        throw new ServiceException(ServiceExceptionEnum.USER_NOT_FOUND);
    }
```
执行结果如下:
![](https://gitee.com/zacharytse/image/raw/master/img/20201108185804.png)
因为handler执行时抛出了异常，所以postHandler()都不会执行
# 跨域问题
## 什么是跨域
因为前后端分离后，前端和后端域名不统一了，所以会存在跨域的问题。例如前端在www.xxx.com的域名下，而后端在www.aaa.com的域名下。
## 解决方案
基于springmvc来解决的话，有3种方案：
1. 使用@CrossCors注解，配置每个API接口
2. 使用CorsRegistry.java注册表，配置每个API接口
3. 使用CorsFilter.java过滤器，处理跨域请求

### @CrossCors
```java{.line-numbers}
package com.xcq.demo.controller;

import com.xcq.demo.domain.UserVO;
import org.springframework.web.bind.annotation.CrossOrigin;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.UUID;

@RestController
@RequestMapping("/test")
@CrossOrigin(origins = "*", allowCredentials = "true") //允许所有来源，允许发送Cookie
public class TestController {

    @GetMapping("/get")
    @CrossOrigin(allowCredentials = "false")//允许所有来源，不允许发送Cookie
    public UserVO get() {
        return new UserVO().setId(1).setUsername(UUID.randomUUID().toString());
    }
}
```
### CorsRegistry.java
在SpringMVCConfiguration.java中重写方法addCorsMappings()
```java{.line-numbers}
@Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")//匹配所有url，相当于全局配置
                .allowedOrigins("*")//允许所有请求来源
                .allowCredentials(true)//允许发送Cookie
                .allowedMethods("*")//允许所有请求method
                .allowedHeaders("*")//允许所有请求Header
                .maxAge(1800L);//有效期1800s,2小时
    }
```
### CorsFilter.java
```java{.line-numbers}
// SpringMVCConfiguration.java

@Bean
public FilterRegistrationBean<CorsFilter> corsFilter() {
    // 创建 UrlBasedCorsConfigurationSource 配置源，类似 CorsRegistry 注册表
    UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
    // 创建 CorsConfiguration 配置，相当于 CorsRegistration 注册信息
    CorsConfiguration config = new CorsConfiguration();
    config.setAllowedOrigins(Collections.singletonList("*")); // 允许所有请求来源
    config.setAllowCredentials(true); // 允许发送 Cookie
    config.addAllowedMethod("*"); // 允许所有请求 Method
    config.setAllowedHeaders(Collections.singletonList("*")); // 允许所有请求 Header
    // config.setExposedHeaders(Collections.singletonList("*")); // 允许所有响应 Header
    config.setMaxAge(1800L); // 有效期 1800 秒，2 小时
    source.registerCorsConfiguration("/**", config);
    // 创建 FilterRegistrationBean 对象
    FilterRegistrationBean<CorsFilter> bean = new FilterRegistrationBean<>(
            new CorsFilter(source)); // 创建 CorsFilter 过滤器
    bean.setOrder(0); // 设置 order 排序。这个顺序很重要哦，为避免麻烦请设置在最前
    return bean;
}
```
# HttpMessageConverter 消息转换器
在 Spring MVC 中，可以使用 @RequestBody 和 @ResponseBody 两个注解，分别完成请求报（内容）到对象和**对象到响应报文（内容）**的转换，底层这种灵活的消息转换机制，就是 Spring 3.x 中新引入的 HttpMessageConverter ，即消息转换器机制。

HttpMessageConverter接口代码如下:
```java{.line-numbers}
// HttpMessageConverter.java

// 是否能够读取指定的 mediaType 内容类型，转换成对应的 clazz 对象
boolean canRead(Class<?> clazz, @Nullable MediaType mediaType);
// 读取请求内容，转换成 clazz 对象
T read(Class<? extends T> clazz, HttpInputMessage inputMessage)
			throws IOException, HttpMessageNotReadableException;

// 是否能够将 clazz 对象，序列化成 mediaType 内容类型
boolean canWrite(Class<?> clazz, @Nullable MediaType mediaType);
// 将 clazz 对象，序列化成 contentType 内容类型，写入到响应。
void write(T t, @Nullable MediaType contentType, HttpOutputMessage outputMessage)
			throws IOException, HttpMessageNotWritableException;

// 获得 HttpMessageConverter 能够支持的内容类型。
List<MediaType> getSupportedMediaTypes();
```
具体流程如下:
- 在**请求**时，在请求头Content-type上，表示请求内容(Request body)的内容类型。Springmvc会从自己的HttpMessageConverter数组中，通过canRead()方法，判断是否能够读取指定的mediaType内容类型，并将其转换为clazz对象。如果可以的话，则调用read()方法，读取请求内容，转换成clazz对象
- 在**响应**时，在请求头Accept上，表示请求内容(Response body)的内容类型。springmvc会从自己的HttpMessageConverter数组中，通过canWrite()方法，判断是否能够将clazz对象序列化成mediaType内容类型，如果可以，则调用write()方法，将clazz对象，序列化为convertType内容类型，写入到响应中。

在springmvc的配置文件中重定义configureMessageConverters()方法
```java{.line-numbers}
@Override
    public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
        //增加xml消息转换器
        Jackson2ObjectMapperBuilder xmlBuilder = Jackson2ObjectMapperBuilder.xml();
        xmlBuilder.indentOutput(true);
        converters.add(new MappingJackson2HttpMessageConverter(xmlBuilder.build()));
    }
```
测试接口
```java{.line-numbers}
@PostMapping(value = "/add",
            // ↓ 增加 "application/xml"、"application/json" ，针对 Content-Type 请求头
            consumes = {MediaType.APPLICATION_XML_VALUE, MediaType.APPLICATION_JSON_VALUE},
            // ↓ 增加 "application/xml"、"application/json" ，针对 Accept 请求头
            produces = {MediaType.APPLICATION_XML_VALUE, MediaType.APPLICATION_JSON_VALUE})
    public UserVO add(@RequestBody UserVO user) {
        return user;
    }
```
使用postman进行接口测试
![](https://gitee.com/zacharytse/image/raw/master/img/20201108194522.png)