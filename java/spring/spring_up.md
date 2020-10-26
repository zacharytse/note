[TOC]
#spring概述
是一种分层的java se/ee应用的full-stack轻量级开源框架。
#问题
实际开发中怎么做到编译器不依赖，运行时才依赖
```java{.line-numbers}
public class JdbcDemo01 {
    public static void main(String[] args) throws SQLException {
        //1.注册驱动
        //DriverManager.registerDriver(new com.mysql.cj.jdbc.Driver());
        Class.forName("com.mysql.cj.jdbc.Driver");//实现了解耦，不会在编译器因为缺少这个类而报错，报错会推迟到运行时
        //2.获取连接
        Connection conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/spring?serverTimezone=UTC",
                "root","xie2481");
        //3.获取操作数据库的预处理对象
        PreparedStatement pstm = conn.prepareStatement("select * from account");
        //4.执行sql语句，得到结果集
        ResultSet rs = pstm.executeQuery();
        //5.遍历结果集
        while(rs.next()){
            System.out.println(rs.getString("name"));
        }
        //6.释放资源
        rs.close();
        pstm.close();
        conn.close();
    }
}
```
解耦的思路:
- 使用反射创建对象，避免使用new关键字
- 通过读取配置文件来获取要创建的对象全限定类名
##工厂模式解耦
> Bean在计算机英语中，有可重用组件的含义

>javabean不是实体类，javabean的范围远大于实体类。用java语言编写的可重用组件
```java{.line-numbers}
//Client.java
package com.xcq.ui;

import com.xcq.factory.BeanFactory;
import com.xcq.service.IAccountService;
import com.xcq.service.impl.AccountServiceImpl;

/**
 * 模拟一个表现层，用于调用业务层
 */
public class Client {

    public static void main(String[] args) {
        IAccountService as = (IAccountService) BeanFactory.getBean("accountService");
        as.saveAccount();
    }
}
```
service层
```java{.line-numbers}
package com.xcq.service;

/**
 * 账户业务层的接口
 */
public interface IAccountService {

    /**
     * 模拟保存
     */
    void saveAccount();
}
```
```java{.line-numbers}
package com.xcq.service.impl;

import com.xcq.dao.IAccountDao;
import com.xcq.dao.impl.AccountDaoImpl;
import com.xcq.factory.BeanFactory;
import com.xcq.service.IAccountService;

/**
 * 账户的业务层实现类
 */
public class AccountServiceImpl implements IAccountService {
    //private IAccountDao accountDao = new AccountDaoImpl();
    private IAccountDao accountDao = (IAccountDao) BeanFactory.getBean("accountDao");
    public void saveAccount() {
        accountDao.saveAccount();
    }
}
```
```java{.line-numbers}
package com.xcq.dao;

/**
 * 账户的持久层接口
 */
public interface IAccountDao {
    /**
     * 模拟保存
     */
    void saveAccount();
}
```
```java{.line-numbers}
package com.xcq.dao.impl;

import com.xcq.dao.IAccountDao;

/**
 * 账户的持久层实现类
 */
public class AccountDaoImpl implements IAccountDao {
    public void saveAccount() {
        System.out.println("保存了账户.........");
    }
}
```
工厂方法
```java{.line-numbers}
package com.xcq.factory;

import java.io.IOException;
import java.io.InputStream;
import java.util.Properties;

/**
 * 创建Bean对象的工厂
 * 就是创建service和dao对象
 * 1.需要一个配置文件来配置service和dao
 *      配置的内容:唯一标志,全限定类名
 * 2.通过读取配置文件中的配置内容，反射创建对象
 * 配置文件可以是xml也可以是properties
 */
public class BeanFactory {
    //定义一个properties对象
    private static Properties props;

    //使用静态代码块为properties对象赋值
    static {
        try {
            //实例化对象
            props = new Properties();
            //获取properties文件的流对象
            InputStream in = BeanFactory.class.getClassLoader().getResourceAsStream("bean.properties");
            props.load(in);
        } catch (IOException e) {
            throw new ExceptionInInitializerError("初始化properties失败");
        }
    }

    /**
     * 根据bean的名称获取bean对象
     * @param beanName
     * @return
     */
    public static Object getBean(String beanName){
        Object bean = null;
        try {
            String beanPath = props.getProperty(beanName);
            bean = Class.forName(beanPath).newInstance();
        }catch (Exception e){
            e.printStackTrace();
        }
        return bean;
    }
}
```
配置文件
```properties
accountService=com.xcq.service.impl.AccountServiceImpl
accountDao=com.xcq.dao.impl.AccountDaoImpl
```
因为service和dao中没有类内变量，所以不需要考虑多线程同时访问同一个变量这种情况，而且单例模式比每次都创建一个新对象效率要高，所以采用单例模式创建对象
单例模式优化factory
```java{.line-numbers}
package com.xcq.factory;

import java.io.IOException;
import java.io.InputStream;
import java.util.Enumeration;
import java.util.HashMap;
import java.util.Map;
import java.util.Properties;

/**
 * 创建Bean对象的工厂
 * 就是创建service和dao对象
 * 1.需要一个配置文件来配置service和dao
 *      配置的内容:唯一标志,全限定类名
 * 2.通过读取配置文件中的配置内容，反射创建对象
 * 配置文件可以是xml也可以是properties
 */
public class BeanFactory {
    //定义一个properties对象
    private static Properties props;

    //定义一个Map,用于存放我们要创建的对象
    private static Map<String,Object> beans;

    //使用静态代码块为properties对象赋值
    static {
        try {
            //实例化对象
            props = new Properties();
            //获取properties文件的流对象
            InputStream in = BeanFactory.class.getClassLoader().getResourceAsStream("bean.properties");
            props.load(in);
            //实例化容器
            beans = new HashMap<>();
            //取出配置文件中所有的key
            Enumeration keys = props.keys();
            //遍历枚举
            while(keys.hasMoreElements()){
                String key = keys.nextElement().toString();
                //根据key获取value
                String beanPath = props.getProperty(key);
                //反射创建对象
                Object value = Class.forName(beanPath).newInstance();
                beans.put(key,value);
            }
        } catch (IOException | ClassNotFoundException e) {
            throw new ExceptionInInitializerError("初始化properties失败");
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        } catch (InstantiationException e) {
            e.printStackTrace();
        }
    }

    /**
     * 根据bean的名称获取bean对象
     * @param beanName
     * @return
     */
    /*public static Object getBean(String beanName){
        Object bean = null;
        try {
            String beanPath = props.getProperty(beanName);
            bean = Class.forName(beanPath).newInstance();
        }catch (Exception e){
            e.printStackTrace();
        }
        return bean;
    }*/

    /**
     * 根据bean的名称获取bean对象
     * @param beanName
     * @return
     */
    public static Object getBean(String beanName){
        return beans.get(beanName);
    }
}
```
#IOC的概念和spring中的IOC
先看这段代码
```java{.line-numbers}
public class AccountServiceImpl implements IAccountService {
    //private IAccountDao accountDao = new AccountDaoImpl();
    private IAccountDao accountDao = (IAccountDao) BeanFactory.getBean("accountDao");
    public void saveAccount() {
        accountDao.saveAccount();
    }
}
```
本来AccountServiceImpl可以选择用new直接创建自己需要的持久层对象，但现在它把这个创建持久层对象的权力交给了BeanFactory，AccountServiceImpl失去了创建持久层对象的权力，所以叫控制反转。作用也就是削减程序的耦合
**实例如下**
dao和service代码不变，去掉factory包，对象的创建交由spring来完成
配置文件如下:
```xml{.line-numbers}
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">
    <!--把对象的创建交给spring来管理-->
    <bean id="accountService" class="com.xcq.service.impl.AccountServiceImpl"></bean>
    <bean id="accountDao" class="com.xcq.dao.impl.AccountDaoImpl"></bean>
</beans>
```
使用如下
```java{.line-numbers}
package com.xcq.ui;

import com.xcq.dao.IAccountDao;
import com.xcq.service.IAccountService;
import com.xcq.service.impl.AccountServiceImpl;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

/**
 * 模拟一个表现层，用于调用业务层
 */
public class Client {

    /**
     * 获取spring容器的IOC,并根据id获取对象
     * @param args
     */
    public static void main(String[] args) {
        //获取核心容器对象
        ApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml");
        //根据id获取bean对象
        IAccountService as = (IAccountService) ac.getBean("accountService");
        IAccountDao adao = ac.getBean("accountDao",IAccountDao.class);

        System.out.println(as);
        System.out.println(adao);

    }
}
```
ApplicationContext的三个常用实现类
- ClassPathXmlApplicationContext:它可以加载类路径下的配置文件，要求配置文件必须在类路径下，不在的话，加载不了(更常用)
- FileSystemXmlApplicationContext:它可以加载磁盘任意路径下的配置文件(必须有访问权限)
- AnnotationConfigApplicationContext:它是用于读取注解创建容器
```java{.line-numbers}
package com.xcq.ui;

import com.xcq.dao.IAccountDao;
import com.xcq.service.IAccountService;
import com.xcq.service.impl.AccountServiceImpl;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import org.springframework.context.support.FileSystemXmlApplicationContext;

/**
 * 模拟一个表现层，用于调用业务层
 */
public class Client {

    /**
     * 获取spring容器的IOC,并根据id获取对象
     * @param args
     */
    public static void main(String[] args) {
        //获取核心容器对象
        ApplicationContext ac = new FileSystemXmlApplicationContext("F:\\springlearn\\spring-ioc\\src\\main\\resources\\bean.xml");
        //根据id获取bean对象
        IAccountService as = (IAccountService) ac.getBean("accountService");
        IAccountDao adao = ac.getBean("accountDao",IAccountDao.class);

        System.out.println(as);
        System.out.println(adao);

    }
}
```
##核心容器的两个接口引发出的问题
- ApplicationContext:它在创建核心容器时，创建对象采取的策略是立即加载的方式，也就是说一读取完配置文件马上就创建配置文件中指定的对象。(单例对象适用)
- BeanFactory:它在构建核心容器时，创建对象采取的策略是延迟加载，也就是什么时候根据id获取对象了，什么时候才创建对象(多例对象适用)
##spring对bean的管理细节
###创建bean的三种方式
```xml
<!--第一种方式：使用默认构造函数创建
        在spring的配置文件中使用bean标签，配以id和class属性之后，且没有其他属性和标签时，
        采用的就是默认构造函数创建bean对象，此时如果类中没有默认构造函数，则对象无法创建-->
    <bean id="accountService" class="com.xcq.service.impl.AccountServiceImpl"></bean>
```
```xml
<!--第二种方式，使用普通工厂中的方法创建对象(使用某个类中的方法创建对象，并存入spring容器)-->
    <bean id="instanceFactory" class="com.xcq.factory.InstanceFactory"></bean>
    <bean id="accountService" factory-bean="instanceFactory" factory-method="getAccountService"></bean>
```
```java{.line-numbers}
package com.xcq.factory;

import com.xcq.service.IAccountService;
import com.xcq.service.impl.AccountServiceImpl;

/**
 * 模拟一个工厂类，该类可能是存在于jar包中的，我们无法通过修改源码的方式来提供默认构造函数
 */
public class InstanceFactory {
    public IAccountService getAccountService(){
        return new AccountServiceImpl();
    }
}

```
```xml
<!--第三种方式，使用静态工厂中的静态方法创建对象(使用某个类中的静态方法创建对象，并存入spring容器-->
    <bean id="accountService" class="com.xcq.factory.StaticFactory" factory-method="getAccountService"></bean>
```
```java{.line-numbers}
package com.xcq.factory;

import com.xcq.service.IAccountService;
import com.xcq.service.impl.AccountServiceImpl;

/**
 * 模拟一个工厂类，该类可能是存在于jar包中的，我们无法通过修改源码的方式来提供默认构造函数
 */
public class StaticFactory {
    public static IAccountService getAccountService(){
        return new AccountServiceImpl();
    }
}
```
###bean的作用范围调整
bean标签的scope属性，用于指定bean的作用范围取值：常用的是单例和多例
- singleton:单例(默认值)
- prototype:多例
- request:作用于web应用的请求范围
- session:作用于web应用的会话范围
- global-session:作用于集群环境的会话范围(全局会话范围),当不是集群环境时，它就是session
###bean对象的生命周期
> 单例对象
- 出生:当容器创建时，对象出生
- 活着:只要容器还在，对象一直活着
- 死亡：容器销毁，对象消亡
总结:单例对象的生命周期和容器相同
> 多例对象
- 出生:当我们使用对象时spring框架为我们创建
- 活着：对象在使用过程中就一直活着
- 死亡:当对象长时间不用且没有别的对象引用时，由java的垃圾回收器回收
#依赖注入(Dependency Injection)
对对象依赖关系的维护称之为依赖注入，可注入的数据有3类(这些数据不能是经常变化的)
- 基本类型和String
- 其他的bean类型(在配置文件或者注解配置过的bean)
- 复杂/集合类型
注入的方法也有3种
- 使用构造函数提供(一般不用)
```xml{.line-numbers}
<!--构造函数注入
    type:用于指定要注入的数据的数据类型，该数据类型也是构造函数中某个或者某些参数的类型
    index:用于指定要注入的数据给构造函数中指定索引位置的参数赋值
    name:用于指定给构造函数中指定名称的参数赋值
    value:用于提供基本类型和string类型的数据
    ref:用于指定其他的bean类型数据，它指的就是在spring的ioc核心容器中出现过的bean对象-->
    <bean name="accountService" class="com.xcq.service.impl.AccountServiceImpl">
        <constructor-arg name="name" value="test"></constructor-arg>
        <constructor-arg name="age" value="18"></constructor-arg>
        <constructor-arg name="birthday" ref="now"></constructor-arg>
    </bean>

    <!--配置一个日期对象-->
    <bean id="now" class="java.util.Date"></bean>
```
> 特点：在获取bean对象时，注入数据是必须的操作，否则对象无法创建成功

> 弊端:改变了bean对象的实例化方式，使我们在创建对象时，如果用不到这些数据，也必须提供
- 使用set方法提供(更常用)
```xml{.line-numbers}
<!--set方法注入
    name:指定注入时要调用的set方法名称
    value:用于提供基本类型和string类型的数据
    ref:用于指定其他的bean类型数据，它指的就是在spring的ioc核心容器中出现过的bean对象-->
    <bean name="accountService2" class="com.xcq.service.impl.AccountServiceImpl2">
        <property name="name" value="test"></property>
        <property name="age" value="18"></property>
        <property name="birthday" ref="now"></property>
    </bean>
```
```xml{.line-numbers}
 <!--复杂类型的注入/集合类型的注入
    用于给list结构集合注入的标签 list,array,set
    用于给map结构注入的标签 props map
    结构相同时，可以互换-->
    <bean name="accountService3" class="com.xcq.service.impl.AccountServiceImpl3">
        <property name="myStrs">
            <array>
                <value>AAA</value>
                <value>BBB</value>
                <value>CCC</value>
            </array>
        </property>
        <property name="myLists">
            <list>
                <value>AAA</value>
                <value>BBB</value>
                <value>CCC</value>
            </list>
        </property>
        <property name="mySet">
            <set>
                <value>AAA</value>
                <value>BBB</value>
                <value>CCC</value>
            </set>
        </property>
        <property name="myMap">
            <map>
                <entry key="test-a" value="aaa"></entry>
                <entry key="test-b">
                    <value>bbb</value>
                </entry>
            </map>
        </property>
        <property name="myProps">
            <props>
                <prop key="test-c">ccc</prop>
                <prop key="test-d">ddd</prop>
            </props>
        </property>
    </bean>
```
> 特点:创建对象时没有明确的限制，可以直接使用默认构造函数

> 弊端:如果某个成员必须有值，则获取对象时set方法没有执行
##spring中ioc的常用注解
1. @Component
   作用：用于把当前对象存入spring容器中
   属性：value:指定bean的id，当我们不写时，它的默认值时当前类名，且首字母改小写
```xml{.line-numbers}
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd">
    <!--告知spring创建容器时所要扫描的包-->
    <context:component-scan base-package="com.xcq"></context:component-scan>
</beans>
```
```java{.line-numbers}
package com.xcq.service.impl;

import com.xcq.dao.IAccountDao;
import com.xcq.service.IAccountService;
import org.springframework.stereotype.Component;

import java.util.Date;
/**
 * 账户的业务层实现类
 */
@Component("accountService")
public class AccountServiceImpl implements IAccountService {
    private IAccountDao accountDao;
    public void saveAccount() {
        //System.out.println("service中的saveAccount方法执行了。。。。。");
        accountDao.saveAccount();
    }
}
```
```java{.line-numbers}
ApplicationContext ac = new FileSystemXmlApplicationContext("F:\\springlearn\\spring-ioc\\src\\main\\resources\\bean.xml");
        //根据id获取bean对象
        IAccountService as = (IAccountService) ac.getBean("accountService");
        System.out.println(as);
```
2. @Controller(表现层)
3. @Servie(业务层)
4. Repository(持久层)

上述三个是spring框架为我们提供明确的三层使用的注解，方便做分层
5. @Autowired
   自动按照类型注入，只要容器中有**唯一**的一个bean对象类型和要注入的变量类型匹配，就可以注入成功。**有多个类型匹配时**， 可以出现在变量上，也可以是方法上。在使用注解时，set方法就不是必须的了
```java{.line-numbers}
@Component("accountService")
public class AccountServiceImpl implements IAccountService {

    @Autowired
    private IAccountDao accountDao;

    public void saveAccount() {
        //System.out.println("service中的saveAccount方法执行了。。。。。");
        accountDao.saveAccount();
    }
}
```
6. @Qualifier
   按照类中注入的基础之上再按照名称注入
   属性：value,用于指定注入bean的id
```java{.line-numbers}
package com.xcq.service.impl;

import com.xcq.dao.IAccountDao;
import com.xcq.service.IAccountService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.stereotype.Component;

import java.util.Date;
/**
 * 账户的业务层实现类
 */
@Component("accountService")
public class AccountServiceImpl implements IAccountService {

    @Autowired
    @Qualifier("accountDao")
    private IAccountDao accountDao;

    public void saveAccount() {
        //System.out.println("service中的saveAccount方法执行了。。。。。");
        accountDao.saveAccount();
    }
}
```
7. @Resource
   直接按照bean的id注入，可以独立使用。属性:name，用于指定bean的id
```java{.line-numbers}
package com.xcq.service.impl;

import com.xcq.dao.IAccountDao;
import com.xcq.service.IAccountService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.stereotype.Component;

import javax.annotation.Resource;
import java.util.Date;
/**
 * 账户的业务层实现类
 */
@Component("accountService")
public class AccountServiceImpl implements IAccountService {

    /*@Autowired
    @Qualifier("accountDao1")*/
    @Resource(name = "accountDao1")
    private IAccountDao accountDao;

    public void saveAccount() {
        //System.out.println("service中的saveAccount方法执行了。。。。。");
        accountDao.saveAccount();
    }
}
```
**集合类型的注入只能通过xml实现**
8. @Value
   作用：用于注入基本类型和string类型的数据
   属性：value,用于指定数据的值。它可以使用spring中的spel(也就是el表达式)
   spel的写法:${表达式}
9. @Scope
    用于指定bean的作用范围
    属性：value指定范围的取值，常用取值：singleton,prototype
```java{.line-numbers}
package com.xcq.service.impl;

import com.xcq.dao.IAccountDao;
import com.xcq.service.IAccountService;
import org.springframework.context.annotation.Scope;
import org.springframework.stereotype.Component;

import javax.annotation.Resource;

/**
 * 账户的业务层实现类
 */
@Component("accountService")
@Scope("prototype")
public class AccountServiceImpl implements IAccountService {

    /*@Autowired
    @Qualifier("accountDao1")*/
    @Resource(name = "accountDao1")
    private IAccountDao accountDao;

    public void saveAccount() {
        //System.out.println("service中的saveAccount方法执行了。。。。。");
        accountDao.saveAccount();
    }
}
```
10. PreDestroy(了解)
    作用：用于指定销毁方法
11. PostConstruct(了解)
    作用：用于指定初始化方法
## 案例使用xml方式和注解方式实现单表的crud操作
持久层技术选择：dbutils
业务层代码如下:
```java{.line-numbers}
package com.xcq.service;

import com.xcq.domain.Account;

import java.util.List;
/**
 * 账户业务层的接口
 */
public interface IAccountService {

    /**
     * 查询所有
     * @return
     */
    List<Account> findAllAccount();

    /**
     * 查询一个
     * @return
     */
    Account findAccountById(Integer id);

    /**
     * 保存操作
     * @param account
     */
    void saveAccount(Account account);

    /**
     * 更新操作
     * @param account
     */
    void updateAccount(Account account);

    /**
     * 删除用户
     * @param id
     */
    void deleteAccount(Integer id);
}
```
```java{.line-numbers}
package com.xcq.service.impl;

import com.xcq.dao.IAccountDao;
import com.xcq.domain.Account;
import com.xcq.service.IAccountService;
import org.springframework.context.annotation.Scope;
import org.springframework.stereotype.Component;

import javax.annotation.Resource;
import java.util.List;

/**
 * 账户的业务层实现类
 */
public class AccountServiceImpl implements IAccountService {

    private IAccountDao accountDao;

    public void setAccountDao(IAccountDao accountDao) {
        this.accountDao = accountDao;
    }


    public List<Account> findAllAccount() {
        return accountDao.findAllAccount();
    }

    public Account findAccountById(Integer id) {
        return accountDao.findAccountById(id);
    }

    public void saveAccount(Account account) {
        accountDao.saveAccount(account);
    }

    public void updateAccount(Account account) {
        accountDao.updateAccount(account);
    }

    public void deleteAccount(Integer id) {
        accountDao.deleteAccount(id);
    }
}
```
持久层代码如下:
```java{.line-numbers}
package com.xcq.dao;

import com.xcq.domain.Account;

import java.util.List;

/**
 * 账户的持久层接口
 */
public interface IAccountDao {
    /**
     * 查询所有
     * @return
     */
    List<Account> findAllAccount();

    /**
     * 查询一个
     * @return
     */
    Account findAccountById(Integer id);

    /**
     * 保存操作
     * @param account
     */
    void saveAccount(Account account);

    /**
     * 更新操作
     * @param account
     */
    void updateAccount(Account account);

    /**
     * 删除用户
     * @param id
     */
    void deleteAccount(Integer id);
}
```
```java{.line-numbers}
package com.xcq.dao.impl;

import com.xcq.dao.IAccountDao;
import com.xcq.domain.Account;
import org.apache.commons.dbutils.QueryRunner;
import org.apache.commons.dbutils.handlers.BeanHandler;
import org.apache.commons.dbutils.handlers.BeanListHandler;

import java.util.List;

public class IAccountDaoImpl implements IAccountDao {

    private QueryRunner runner;

    public void setRunner(QueryRunner runner) {
        this.runner = runner;
    }

    public List<Account> findAllAccount() {
        try {
            return runner.query("select * from account",new BeanListHandler<Account>(Account.class));
        }catch (Exception e){
            throw new RuntimeException(e);
        }
    }

    public Account findAccountById(Integer id) {
        try {
            return runner.query("select * from account where id = ?",
                    new BeanHandler<Account>(Account.class)
                    ,id);
        }catch (Exception e){
            throw new RuntimeException(e);
        }
    }

    public void saveAccount(Account account) {
        try {
            runner.update("insert  into account(name,money)values (?,?)",account.getName(),account.getMoney());
        }catch (Exception e){
            throw new RuntimeException(e);
        }
    }

    public void updateAccount(Account account) {
        try {
            runner.update("update account set name=?,money=? where id = ?",account.getName(),
                    account.getMoney(),account.getId());
        }catch (Exception e){
            throw new RuntimeException(e);
        }
    }

    public void deleteAccount(Integer id) {
        try {
            runner.update("delete from account where id=?",id);
        }catch (Exception e){
            throw new RuntimeException(e);
        }
    }
}
```
配置文件如下：
```xml{.line-numbers}
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">
    <!--业务层对象-->
    <bean id="accountService" class="com.xcq.service.impl.AccountServiceImpl">
        <!--注入的是dao对象-->
        <property name="accountDao" ref="accountDao"></property>
    </bean>

    <!--配置dao对象-->
    <bean id="accountDao" class="com.xcq.dao.impl.IAccountDaoImpl">
        <!--注入QueryRunner-->
        <property name="runner" ref="runner"></property>
    </bean>

    <!--配置QueryRunner对象-->
    <bean id="runner" class="org.apache.commons.dbutils.QueryRunner" scope="prototype">
        <!--注入数据源-->
        <constructor-arg name="ds" ref="dataSource"></constructor-arg>
    </bean>

    <!--配置数据源-->
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <!--连接数据库的必备信息-->
        <property name="driverClass" value="com.mysql.jdbc.Driver"></property>
        <property name="jdbcUrl" value="jdbc:mysql://localhost:3306/spring?serverTimezone=UTC"></property>
        <property name="user" value="root"></property>
        <property name="password" value="xie2481"></property>
    </bean>
</beans>
```
单元测试如下
```java{.line-numbers}
package com.xcq.test;

import com.xcq.domain.Account;
import com.xcq.service.IAccountService;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import java.util.List;
/**
 * 使用junit单元测试我们的配置
 */
public class AccountServiceTest {

    @Test
    public void testFindAll(){
        //1获取容器
        ApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml");
        IAccountService accountService = ac.getBean("accountService",
                IAccountService.class);
        List<Account> accounts = accountService.findAllAccount();
        for(Account account : accounts){
            System.out.println(account);
        }
    }

    @Test
    public void testFindOne(){
        ApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml");
        IAccountService accountService = ac.getBean("accountService",
                IAccountService.class);
        Account account = accountService.findAccountById(1);
        System.out.println(account);
    }

    @Test
    public void testSave(){
        Account account = new Account();
        account.setName("test");
        account.setMoney(12.0f);
        ApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml");
        IAccountService accountService = ac.getBean("accountService",
                IAccountService.class);
        accountService.saveAccount(account);
    }

    @Test
    public void testUpdate(){
        ApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml");
        IAccountService accountService = ac.getBean("accountService",
                IAccountService.class);
        Account account = accountService.findAccountById(4);
        account.setMoney(245.0f);
        //System.out.println(account.getId());
        accountService.updateAccount(account);
    }

    @Test
    public void testDelete(){
        ApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml");
        IAccountService accountService = ac.getBean("accountService",
                IAccountService.class);
        accountService.deleteAccount(4);
    }
}
```
## 改造基于注解的IOC案例，使用纯注解的方式实现
```java{.line-numbers}
package com.xcq.service.impl;

import com.xcq.dao.IAccountDao;
import com.xcq.domain.Account;
import com.xcq.service.IAccountService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Scope;
import org.springframework.stereotype.Component;
import org.springframework.stereotype.Service;

import javax.annotation.Resource;
import java.util.List;

/**
 * 账户的业务层实现类
 */
@Service("accountService")
public class AccountServiceImpl implements IAccountService {


    @Autowired
    private IAccountDao accountDao;

    public List<Account> findAllAccount() {
        return accountDao.findAllAccount();
    }

    public Account findAccountById(Integer id) {
        return accountDao.findAccountById(id);
    }

    public void saveAccount(Account account) {
        accountDao.saveAccount(account);
    }

    public void updateAccount(Account account) {
        accountDao.updateAccount(account);
    }

    public void deleteAccount(Integer id) {
        accountDao.deleteAccount(id);
    }
}
```
```java{.line-numbers}
package com.xcq.dao.impl;

import com.xcq.dao.IAccountDao;
import com.xcq.domain.Account;
import org.apache.commons.dbutils.QueryRunner;
import org.apache.commons.dbutils.handlers.BeanHandler;
import org.apache.commons.dbutils.handlers.BeanListHandler;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Repository;

import java.util.List;

@Repository("accountDao")
public class IAccountDaoImpl implements IAccountDao {

    @Autowired
    private QueryRunner runner;

    public void setRunner(QueryRunner runner) {
        this.runner = runner;
    }

    public List<Account> findAllAccount() {
        try {
            return runner.query("select * from account",new BeanListHandler<Account>(Account.class));
        }catch (Exception e){
            throw new RuntimeException(e);
        }
    }

    public Account findAccountById(Integer id) {
        try {
            return runner.query("select * from account where id = ?",
                    new BeanHandler<Account>(Account.class)
                    ,id);
        }catch (Exception e){
            throw new RuntimeException(e);
        }
    }

    public void saveAccount(Account account) {
        try {
            runner.update("insert  into account(name,money)values (?,?)",account.getName(),account.getMoney());
        }catch (Exception e){
            throw new RuntimeException(e);
        }
    }

    public void updateAccount(Account account) {
        try {
            runner.update("update account set name=?,money=? where id = ?",account.getName(),
                    account.getMoney(),account.getId());
        }catch (Exception e){
            throw new RuntimeException(e);
        }
    }

    public void deleteAccount(Integer id) {
        try {
            runner.update("delete from account where id=?",id);
        }catch (Exception e){
            throw new RuntimeException(e);
        }
    }
}
```
```xml{.line-numbers}
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd">
    <!--告知spring在创建容器时，要扫描的包-->
    <context:component-scan base-package="com.xcq"></context:component-scan>

    <!--配置QueryRunner对象-->
    <bean id="runner" class="org.apache.commons.dbutils.QueryRunner" scope="prototype">
        <!--注入数据源-->
        <constructor-arg name="ds" ref="dataSource"></constructor-arg>
    </bean>

    <!--配置数据源-->
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <!--连接数据库的必备信息-->
        <property name="driverClass" value="com.mysql.jdbc.Driver"></property>
        <property name="jdbcUrl" value="jdbc:mysql://localhost:3306/spring?serverTimezone=UTC"></property>
        <property name="user" value="root"></property>
        <property name="password" value="xie2481"></property>
    </bean>
</beans>
```
###改进方法
@Configuration
作用:指定当前类是一个配置类
当配置类作为AnnotationConfigApplicationContext对象创建的参数时，该注解可以省略

@ComponentScan
作用:用于通过注解指定spring在创建容器时，要扫描的包
属性：value,它和basePackages作用是一样的，都是用于指定创建容器时，要扫描的包

@Bean
用于把当前方法的返回值作为bean对象，存入spring的IOC容器中。
属性:name,用于指定bean的id，不写时，默认值是当前方法的名称

@Import
用于导入其他的配置类、
```java
@ComponentScan(basePackages = "com.xcq")
@Import(JdbcConfig.class)
```
属性：value,用于指定其他配置类的字节码，当我们使用import注解之后，有import注解的类就是主(父)配置类,导入的都是子配置类

> 当我们使用注解配置方法时，如果方法有参数，spring框架会去容器中查找有没有可用的bean对象，查找的方式和AutoWired作用是一样的。
```java{.line-numbers}
package config;

import com.mchange.v2.c3p0.ComboPooledDataSource;
import org.apache.commons.dbutils.QueryRunner;
import org.springframework.context.annotation.*;

import javax.sql.DataSource;

/**
 * 该类是一个配置类，它的作用和bean.xml是一样的
 */
@Configuration
@ComponentScan(basePackages = "com.xcq")
public class SpringConfiguration {

    /**
     * 用于创建一个QueryRunner对象
     * @param dataSource
     * @return
     */
    @Bean(name = "runner")
    @Scope("prototype")
    public QueryRunner createQueryRunner(DataSource dataSource){
        return new QueryRunner(dataSource);
    }

    /**
     * 创建数据源对象
     * @return
     */
    @Bean(name="dataSource")
    public DataSource createDataSource(){
        try {
            ComboPooledDataSource ds = new ComboPooledDataSource();
            ds.setDriverClass("com.mysql.jdbc.Driver");
            ds.setJdbcUrl("jdbc:mysql://localhost:3306/spring?serverTimezone=UTC");
            ds.setUser("root");
            ds.setPassword("xie2481");
            return ds;
        }catch (Exception e){
            throw new RuntimeException(e);
        }

    }
}
```
```java{.line-numbers}
 @Test
    public void testFindAll(){
        //1获取容器
        //ApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml");
        ApplicationContext ac = new AnnotationConfigApplicationContext(SpringConfiguration.class);
        IAccountService accountService = ac.getBean("accountService",
                IAccountService.class);
        List<Account> accounts = accountService.findAllAccount();
        for(Account account : accounts){
            System.out.println(account);
        }
    }
```
@PropertySources
用于指定properties文件的位置
```java
@PropertySource("classpath:jdbcConfig.properties")
```
value:指定文件的名称和路径
关键字：classpath,表示类路径下
对Jdbc的配置进一步优化
```java{.line-numbers}
package config;

import com.mchange.v2.c3p0.ComboPooledDataSource;
import org.apache.commons.dbutils.QueryRunner;
import org.springframework.context.annotation.*;

import javax.sql.DataSource;

/**
 * 该类是一个配置类，它的作用和bean.xml是一样的
 */
@Configuration
@ComponentScan(basePackages = "com.xcq")
@Import(JdbcConfig.class)
@PropertySource("classpath:jdbcConfig.properties")
public class SpringConfiguration {


}
```
```java{.line-numbers}
package config;

import com.mchange.v2.c3p0.ComboPooledDataSource;
import org.apache.commons.dbutils.QueryRunner;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Scope;

import javax.sql.DataSource;

public class JdbcConfig {
    @Value("${jdbc.driver}")
    private String driver;

    @Value("${jdbc.url}")
    private String url;

    @Value("${jdbc.username}")
    private String username;

    @Value("${jdbc.password}")
    private String password;

    /**
     * 用于创建一个QueryRunner对象
     * @param dataSource
     * @return
     */
    @Bean(name = "runner")
    @Scope("prototype")
    public QueryRunner createQueryRunner(DataSource dataSource){
        return new QueryRunner(dataSource);
    }

    /**
     * 创建数据源对象
     * @return
     */
    @Bean(name="dataSource")
    public DataSource createDataSource(){
        try {
            ComboPooledDataSource ds = new ComboPooledDataSource();
            ds.setDriverClass(driver);
            ds.setJdbcUrl(url);
            ds.setUser(username);
            ds.setPassword(password);
            return ds;
        }catch (Exception e){
            throw new RuntimeException(e);
        }

    }
}
```
```properties
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/spring?serverTimezone=UTC
jdbc.username=root
jdbc.password=xie2481
```
@Qualifier
```java{.line-numbers}
Bean(name = "runner")
    @Scope("prototype")
    public QueryRunner createQueryRunner(@Qualifier("dataSource") DataSource dataSource){
        return new QueryRunner(dataSource);
    }

    /**
     * 创建数据源对象
     * @return
     */
    @Bean(name="dataSource")
    public DataSource createDataSource(){
        try {
            ComboPooledDataSource ds = new ComboPooledDataSource();
            ds.setDriverClass(driver);
            ds.setJdbcUrl(url);
            ds.setUser(username);
            ds.setPassword(password);
            return ds;
        }catch (Exception e){
            throw new RuntimeException(e);
        }

    }
```
# spring和JUnit的整合
1. 导入spring-test的jar包
2. 使用Junit提供的一个注解把原有的main方法替换，替换成spring提供的，该注解为@RunWith
3. 告知spring的运行器，spring和ioc创建是基于xml还是注解的，并且说明位置。@ContextConfiguration，使用locaiton属性，说明是使用配置文件，使用classes属性，说明是使用注解
```java{.line-numbers}
package com.xcq.test;

import com.xcq.domain.Account;
import com.xcq.service.IAccountService;
import config.SpringConfiguration;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

import java.util.List;

/**
 * 使用junit单元测试我们的配置
 */
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = SpringConfiguration.class)
public class AccountServiceTest {

    @Autowired
    private IAccountService accountService;
    
    @Test
    public void testFindAll(){
        List<Account> accounts = accountService.findAllAccount();
        for(Account account : accounts){
            System.out.println(account);
        }
    }

    @Test
    public void testFindOne(){
        Account account = accountService.findAccountById(1);
        System.out.println(account);
    }

    @Test
    public void testSave(){
        Account account = new Account();
        account.setName("test");
        account.setMoney(12.0f);
        accountService.saveAccount(account);
    }

    @Test
    public void testUpdate(){
        Account account = accountService.findAccountById(4);
        account.setMoney(245.0f);
        //System.out.println(account.getId());
        accountService.updateAccount(account);
    }

    @Test
    public void testDelete(){
        accountService.deleteAccount(4);
    }
}
```
