[TOC]
# get和post
从总体上来说，get和post具有以下两点区别:
- get用于获取信息，是无副作用的，是幂等的且可缓存
- post用于修改服务器上的数据，有副作用，非幂等，不可缓存

## 报文上的区别
这两者在传输上没有区别，在不带参数时，最大的区别是第一行的方法名不同
post的第一行是POST /uri HTTP/1.1 \r\n
get的第一行是GET /uri HTTP/1.1 \r\n
带参数时，get的参数放在url中，post的参数应该在body中
## get方法参数写法是固定的吗？
约定中参数是写在?后面，多个参数用&分割。但我们也可以通过自己解析url，定义自己的get参数的规则
## post比get安全吗
答案是否定的，因为它俩用的都是http协议，http协议是不加密的，所以要保证安全性，要使用https协议
## get方法的长度限制
http协议本身没有对url长度有限制，有限制的是浏览器或者是服务器

# Http响应状态码
响应分为5类
- 信息响应(100-199)
- 成功响应(200-299)
- 重定向(300-399)
- 客户端错误(400-499)
- 服务器错误(500-599)

# cookie和session的区别
1. session是一个抽象的概念，开发者为了实现中断和继续等操作，将user agent和server之间一对一的交互，抽象为会话，进而衍生出了会话状态，也就是session。
2. cookie是一个实际存在的东西，它被http协议定义在了header中
3. session的存在是为了绕开cookie中的各种限制，通常借助cookie本身和后端存储实现，是一种更高级的会话状态实现
4. 因为session的运行依赖于session id，而session id存在于cookie中，所以如果cookie失效或被浏览器禁用，session也会实现

# 多个服务器如何保证session的一致性
1. 多个服务器之间相互同步session。但这样session的同步会占据带宽
2. 把session存到客户端(因为安全性，不常用)
3. 反向代理hash一致性，server重启时，会使得session失效
   1. 四层代理hash，使用用户id做hash，保证同一个ip落到同一个server上
   2. 七层代理hash，使用某些业务属性，如用户id之类的做hash
4. 后端统一集中存储，会增加一次网络调用