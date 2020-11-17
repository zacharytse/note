[TOC]
# 登录模块
## 实现方法
基于Redis实现了单点登录。用Redis模拟session。每次访问会随机产生一个sessionid,并将该id存入到redis中。同时登录后，会将该id以cookie的形式返回给客户端。下次登录时，会利用基于aop的环绕通知实现的LoginAspect检查Redis是否存了这个cookie,如果存了，则直接允许登录
## 流程图
![](https://gitee.com/zacharytse/image/raw/master/img/das.png)

## 实现过程中遇到的问题
在处理cookie跨域问题时，错误地将项目部署地址的url设置为了cookie的domain