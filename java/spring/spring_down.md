[TOC]
# 基于动态代理的crud实例
```java{.line-numbers}
package com.xcq.utils;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

/**
 * 和事务管理相关的工具类，它包含了开启事务，提交事务，回滚事务和释放连接
 */
@Component("transactionManager")
public class TransactionManager {

    @Autowired
    private ConnectionUtils connectionUtils;

    /**
     * 开启事务
     */
    public void beginTransaction(){
        try {
            connectionUtils.getThreadConnection().setAutoCommit(false);
        }catch (Exception e){
            e.printStackTrace();
        }
    }

    /**
     * 提交事务
     */
    public void commit(){
        try {
            connectionUtils.getThreadConnection().commit();
        }catch (Exception e){
            e.printStackTrace();
        }
    }

    /**
     * 回滚事务
     */
    public void rollback(){
        try {
            connectionUtils.getThreadConnection().rollback();
        }catch (Exception e){
            e.printStackTrace();
        }
    }

    /**
     * 释放连接
     */
    public void release(){
        try {
            connectionUtils.getThreadConnection().close();//还回了连接池中
            connectionUtils.removeConnection();
        }catch (Exception e){
            e.printStackTrace();
        }
    }
}
```
```java{.line-numbers}
package com.xcq.utils;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Scope;
import org.springframework.stereotype.Component;

import javax.sql.DataSource;
import java.sql.Connection;

/**
 * 连接工具类，用于从数据源中获取一个连接，并且实现和线程的绑定
 */
@Component("connectionUtils")
@Scope("prototype")
public class ConnectionUtils {

    //这里必须是static,因为ConnectionUtils对象可以创建多个，而每一个ConnectionUtils存储的Connection必须相同，否则Dao使用的连接和Service基于业务使用的连接不是同一个连接，最后会造成事务出错时回滚失败
    private static ThreadLocal<Connection> tl = new ThreadLocal<Connection>();

    @Autowired
    private DataSource dataSource;

    /**
     * 获取当前线程上的连接
     * @return
     */
    public Connection getThreadConnection(){
        //1.先从ThreadLocal上获取
        Connection conn = tl.get();
        try {
            //2.判断当前线程上是否有连接
            if(conn == null){
                //3.从数据源中获取一个连接，并且存入ThreadLocal中
                conn = dataSource.getConnection();
                tl.set(conn);
            }
            //4.返回当前线程上的连接
            return conn;
        }catch (Exception e) {
            throw new RuntimeException(e);
        }
    }

    /**
     * 把连接和线程解绑
     */
    public void removeConnection(){
        tl.remove();
    }
}
```
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

    /**
     * 转账
     * @param sourceName 转出账户名称
     * @param targetName 转入账户名称
     * @param money 转账金额
     */
    void transfer(String sourceName,String targetName,float money);
}
```
```java{.line-numbers}
package com.xcq.service.impl;

import com.xcq.dao.IAccountDao;
import com.xcq.domain.Account;
import com.xcq.service.IAccountService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;

/**
 * 账户的业务层实现类
 */
@Service("accountService")
public class AccountServiceImpl implements IAccountService {


    @Autowired
    private IAccountDao accountDao;

    public List<Account> findAllAccount() {
        List<Account> accounts = accountDao.findAllAccount();
        return accounts;
    }

    public Account findAccountById(Integer id) {
        Account account = accountDao.findAccountById(id);
        return account;
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

    public void transfer(String sourceName, String targetName, float money) {
        //2.执行操作
        //2.1.根据名称查询转出账户
        Account source = accountDao.findAccountByName(sourceName);
        //2.2.根据名称查询转入账户
        Account target = accountDao.findAccountByName(targetName);
        //2.3.转出账户减钱
        source.setMoney(source.getMoney() - money);
        //2.4.转入账户加钱
        target.setMoney(target.getMoney() + money);
        //2.5.更新转出账户
        accountDao.updateAccount(source);
        //int i =  1/0;
        //2.6.更新转入账户
        accountDao.updateAccount(target);
       /* try{
            transactionManager.beginTransaction();

            transactionManager.commit();
        }catch (Exception e){
            transactionManager.rollback();
            throw new RuntimeException(e);
        }finally {
            transactionManager.release();
        }*/


    }
}
```
```java{.line-numbers}
package com.xcq.factory;

import com.xcq.service.IAccountService;
import com.xcq.utils.TransactionManager;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.stereotype.Component;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

/**
 * 用于创建service的代理对象的工厂
 */
@Component("beanFactory")
public class BeanFactory {

    @Autowired
    private IAccountService accountService;

    @Autowired
    private TransactionManager transactionManager;

    @Bean(name = "accountServiceProxy")
    public IAccountService getAccountService(){
        return (IAccountService) Proxy.newProxyInstance(accountService.getClass().getClassLoader(),
                accountService.getClass().getInterfaces(),
                new InvocationHandler() {
                    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                        Object retVal = null;
                        try {
                            //1.开启事务
                            transactionManager.beginTransaction();
                            //2.执行操作
                            retVal = method.invoke(accountService,args);
                            //3.提交事务
                            transactionManager.commit();
                            return retVal;
                        }catch (Exception e){
                            transactionManager.rollback();
                            throw new RuntimeException(e);
                        }finally {
                            //6.释放连接
                            transactionManager.release();
                        }

                    }
                });
    }
}
```
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

    /**
     * 根据名称查询账户
     * @param accountName
     * @return 如果有唯一的一个结果，就返回，没有结果，就返回null
     *         如果结果集超过一个，就抛异常
     */
    Account findAccountByName(String accountName);
}
```
```java{.line-numbers}
package com.xcq.dao.impl;

import com.xcq.dao.IAccountDao;
import com.xcq.domain.Account;
import com.xcq.utils.ConnectionUtils;
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

    @Autowired
    private ConnectionUtils connectionUtils;

    public List<Account> findAllAccount() {
        try {
            return runner.query(connectionUtils.getThreadConnection(),"select * from account",new BeanListHandler<Account>(Account.class));
        }catch (Exception e){
            throw new RuntimeException(e);
        }
    }

    public Account findAccountById(Integer id) {
        try {
            return runner.query(connectionUtils.getThreadConnection(),"select * from account where id = ?",
                    new BeanHandler<Account>(Account.class)
                    ,id);
        }catch (Exception e){
            throw new RuntimeException(e);
        }
    }

    public void saveAccount(Account account) {
        try {
            runner.update(connectionUtils.getThreadConnection(),"insert  into account(name,money)values (?,?)",account.getName(),account.getMoney());
        }catch (Exception e){
            throw new RuntimeException(e);
        }
    }

    public void updateAccount(Account account) {
        try {
            runner.update(connectionUtils.getThreadConnection(),"update account set name=?,money=? where id = ?",account.getName(),
                    account.getMoney(),account.getId());
        }catch (Exception e){
            throw new RuntimeException(e);
        }
    }

    public void deleteAccount(Integer id) {
        try {
            runner.update(connectionUtils.getThreadConnection(),"delete from account where id=?",id);
        }catch (Exception e){
            throw new RuntimeException(e);
        }
    }

    public Account findAccountByName(String accountName) {
        try {
            List<Account> accounts = runner.query(connectionUtils.getThreadConnection(),"select * from account where name = ?",
                    new BeanListHandler<Account>(Account.class)
                    ,accountName);
            if(accounts == null || accounts.size() == 0){
                return null;
            }
            if(accounts.size() > 1){
                throw new RuntimeException("结果集不为1");
            }
            return accounts.get(0);
        }catch (Exception e){
            throw new RuntimeException(e);
        }
    }
}
```
```java{.line-numbers}
package com.xcq.domain;

import java.io.Serializable;

/**
 * 账户实体类
 */
public class Account implements Serializable {
    private Integer id;
    private String name;
    private Float money;

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
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
                "id=" + id +
                ", name='" + name + '\'' +
                ", money=" + money +
                '}';
    }
}
```
# AOP概念
面向切面编程
作用:在运行期间，不修改源码，对原方法进行增强
实现方式：使用动态代理
## AOP相关术语
1. JoinPoint
   业务层中所有的方法
2. PointCut
   被增强的方法
   所有的切入点都是连接点，反之不成立
3. Advice
   拦截到之后，要做的事情
   在环绕通知中，有明确的切入点方法调用
![](https://gitee.com/zacharytse/image/raw/master/img/通知的类型.jpg)
4. Target
   被代理对象
5. Weaving(织入)
   把增强应用到目标对象来创建新的代理对象的过程
6. Proxy
   代理对象
7. Aspect(切面)
   切入点和通知的结合

# 基于xml和注解的AOP配置
```xml{.line-numbers}
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop
        http://www.springframework.org/schema/aop/spring-aop.xsd">

    <!--配置spring的ioc,把service对象配置进来-->
    <bean id="accountService" class="com.xcq.service.impl.AccountServiceImpl"></bean>

    <!--spring中基于xml的aop配置步骤
    1.把通知的bean交给spring管理
    2.使用aop:config标签表明开始aop的配置
    3.使用aop:aspect表明配置切面
        id:给切面提供一个唯一标志
        ref:指定通知类bean的id
    4.在aop:aspect内部使用对应的标签来配置通知的类型
      我们现在的示例是让pringLog方法在切入点方法执行之前执行，
      所以是前置通知
      aop:before表示配置前置通知
      method属性用于指定Logger类中哪个方法是前置通知
      pointcut属性：用于指定切入点表达式，该表达式指的是对哪些业务方法进行增强
      切入点表达式的写法:
      关键字：execution(表达式)
      表达式：
      访问修饰符，返回值，包名.包名.....类名.方法名(参数列表)
      访问修饰符可以省略
      返回值可以使用通配符表示任意返回值
      包名可以使用通配符表示任意包，但是有几级包，就需要写几个*
      包名可以使用..表示当前包及其子包
      类名和方法名都可以用*
      参数列表可以直接写数据类型
      基本类型写名称，引用类型写包名.类名
      可以使用通配符匹配所有参数，但必须有参数
      使用..既可以匹配所有有参数的，也可以匹配无参的
      全通配写法
      * *..*.*(..)-->

    <!--配置Logger类-->
    <bean id="logger" class="com.xcq.utils.Logger"></bean>

    <!--配置aop-->
    <aop:config>
        <!--配置切面-->
        <aop:aspect id="logAdvice" ref="logger">
            <!--配置通知的类型并且建立通知方法和切入点方法的关联-->
            <aop:before method="printLog"
                        pointcut="execution(* com.xcq.service.impl.*.*(..))"></aop:before>
        </aop:aspect>
    </aop:config>
</beans>
```
```java{.line-numbers}
package com.xcq.utils;

/**
 * 用于记录日志的工具类
 */
public class Logger {

    /**
     * 用于打印日志，并且计划让其在切入点方法执行之前执行(切入点方法就是业务层方法)
     */
    public void printLog(){
        System.out.println("Logger类中的log开始打印日志");
    }
}
```
```java{.line-numbers}
package com.xcq.service;

/**
 * 账户的业务层接口
 */
public interface IAccountService {

    /**
     * 模拟保存账户
     */
    void saveAccount();

    /**
     * 模拟更新账户
     * @param i
     */
    void updateAccount(int i);

    /**
     * 删除账户
     * @return
     */
    int deleteAccount();
}
```
```java{.line-numbers}
package com.xcq.service.impl;

import com.xcq.service.IAccountService;

/**
 * 账户的业务层实现类
 */
public class AccountServiceImpl implements IAccountService {
    public void saveAccount() {
        System.out.println("执行了保存");
    }

    public void updateAccount(int i) {
        System.out.println("执行了更新");
    }

    public int deleteAccount() {
        System.out.println("执行了删除");
        return 0;
    }
}
```
## 各种advice
```xml{.line-numbers}
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop
        http://www.springframework.org/schema/aop/spring-aop.xsd">

    <!--配置spring的ioc,把service对象配置进来-->
    <bean id="accountService" class="com.xcq.service.impl.AccountServiceImpl"></bean>

    <!--spring中基于xml的aop配置步骤
    1.把通知的bean交给spring管理
    2.使用aop:config标签表明开始aop的配置
    3.使用aop:aspect表明配置切面
        id:给切面提供一个唯一标志
        ref:指定通知类bean的id
    4.在aop:aspect内部使用对应的标签来配置通知的类型
      我们现在的示例是让pringLog方法在切入点方法执行之前执行，
      所以是前置通知
      aop:before表示配置前置通知
      method属性用于指定Logger类中哪个方法是前置通知
      pointcut属性：用于指定切入点表达式，该表达式指的是对哪些业务方法进行增强
      切入点表达式的写法:
      关键字：execution(表达式)
      表达式：
      访问修饰符，返回值，包名.包名.....类名.方法名(参数列表)
      访问修饰符可以省略
      返回值可以使用通配符表示任意返回值
      包名可以使用通配符表示任意包，但是有几级包，就需要写几个*
      包名可以使用..表示当前包及其子包
      类名和方法名都可以用*
      参数列表可以直接写数据类型
      基本类型写名称，引用类型写包名.类名
      可以使用通配符匹配所有参数，但必须有参数
      使用..既可以匹配所有有参数的，也可以匹配无参的
      全通配写法
      * *..*.*(..)-->

    <!--配置Logger类-->
    <bean id="logger" class="com.xcq.utils.Logger"></bean>

    <!--配置aop-->
    <aop:config>
        <!--配置切面-->
        <aop:aspect id="logAdvice" ref="logger">
            <!--配置通知的类型并且建立通知方法和切入点方法的关联-->
            <aop:before method="beforePrintLog"
                        pointcut="execution(* com.xcq.service.impl.*.*(..))"></aop:before>
            <!--后置通知-->
            <aop:after-returning method="afterPrintLog"
                        pointcut="execution(* com.xcq.service.impl.*.*(..))"></aop:after-returning>
            <aop:after-throwing method="afterThrowingPrintLog"
                        pointcut="execution(* com.xcq.service.impl.*.*(..))"></aop:after-throwing>
            <!--最终通知-->
            <aop:after method="afterFinalPrintLog"
                        pointcut="execution(* com.xcq.service.impl.*.*(..))"></aop:after>
        </aop:aspect>
    </aop:config>
</beans>
```
切入点表达式的优化
```xml{.line-numbers}
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop
        http://www.springframework.org/schema/aop/spring-aop.xsd">

    <!--配置spring的ioc,把service对象配置进来-->
    <bean id="accountService" class="com.xcq.service.impl.AccountServiceImpl"></bean>

    <!--spring中基于xml的aop配置步骤
    1.把通知的bean交给spring管理
    2.使用aop:config标签表明开始aop的配置
    3.使用aop:aspect表明配置切面
        id:给切面提供一个唯一标志
        ref:指定通知类bean的id
    4.在aop:aspect内部使用对应的标签来配置通知的类型
      我们现在的示例是让pringLog方法在切入点方法执行之前执行，
      所以是前置通知
      aop:before表示配置前置通知
      method属性用于指定Logger类中哪个方法是前置通知
      pointcut属性：用于指定切入点表达式，该表达式指的是对哪些业务方法进行增强
      切入点表达式的写法:
      关键字：execution(表达式)
      表达式：
      访问修饰符，返回值，包名.包名.....类名.方法名(参数列表)
      访问修饰符可以省略
      返回值可以使用通配符表示任意返回值
      包名可以使用通配符表示任意包，但是有几级包，就需要写几个*
      包名可以使用..表示当前包及其子包
      类名和方法名都可以用*
      参数列表可以直接写数据类型
      基本类型写名称，引用类型写包名.类名
      可以使用通配符匹配所有参数，但必须有参数
      使用..既可以匹配所有有参数的，也可以匹配无参的
      全通配写法
      * *..*.*(..)-->

    <!--配置Logger类-->
    <bean id="logger" class="com.xcq.utils.Logger"></bean>

    <!--配置aop-->
    <aop:config>
        <!--配置切面-->
        <aop:aspect id="logAdvice" ref="logger">
            <!--配置通知的类型并且建立通知方法和切入点方法的关联-->
            <aop:before method="beforePrintLog"
                        pointcut-ref="pt1"></aop:before>

            <aop:after-returning method="afterPrintLog"
                        pointcut-ref="pt1"></aop:after-returning>
            <aop:after-throwing method="afterThrowingPrintLog"
                        pointcut-ref="pt1"></aop:after-throwing>
            <aop:after method="afterFinalPrintLog"
                        pointcut-ref="pt1"></aop:after>
            <!--配置切入点表达式 id属性用于指定表达式的唯一标识 expression指定表达式内容
            此标签写在aspect中，只能当前切面使用，它也可以写在aop:aspect外面，且必须要写在aspect之前，此时就变成了所有切面可用-->
            <aop:pointcut id="pt1" expression="execution(* com.xcq.service.impl.*.*(..))"/>
        </aop:aspect>
    </aop:config>
</beans>
```
### 环绕通知
Spring提供了一个接口：ProceedingJoinPoint,该接口有一个方法proceed(),此方法就相当于明确调用切入点方法，该接口可以作为环绕通知的方法参数，在程序执行时，spring框架会为我们提供该接口的实现类供我们使用
```java{.line-numbers}
 /**
     * 环绕通知
     */
    public Object aroundPrintLog(ProceedingJoinPoint proceedingJoinPoint){
        Object retVal = null;
        try {
            System.out.println("Logger类中的aroundPrintLog开始打印日志...前置");
            Object[] args = proceedingJoinPoint.getArgs();//得到方法执行所需的参数
            retVal = proceedingJoinPoint.proceed(args);//明确调用业务层方法
            System.out.println("Logger类中的aroundPrintLog开始打印日志...后置");
            return retVal;
        }catch (Throwable t){
            System.out.println("Logger类中的aroundPrintLog开始打印日志...异常");
        }finally {
            System.out.println("Logger类中的aroundPrintLog开始打印日志...最终");
        }
        return null;
    }
```
```xml{.line-numbers}
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop
        http://www.springframework.org/schema/aop/spring-aop.xsd">

    <!--配置spring的ioc,把service对象配置进来-->
    <bean id="accountService" class="com.xcq.service.impl.AccountServiceImpl"></bean>

    <!--spring中基于xml的aop配置步骤
    1.把通知的bean交给spring管理
    2.使用aop:config标签表明开始aop的配置
    3.使用aop:aspect表明配置切面
        id:给切面提供一个唯一标志
        ref:指定通知类bean的id
    4.在aop:aspect内部使用对应的标签来配置通知的类型
      我们现在的示例是让pringLog方法在切入点方法执行之前执行，
      所以是前置通知
      aop:before表示配置前置通知
      method属性用于指定Logger类中哪个方法是前置通知
      pointcut属性：用于指定切入点表达式，该表达式指的是对哪些业务方法进行增强
      切入点表达式的写法:
      关键字：execution(表达式)
      表达式：
      访问修饰符，返回值，包名.包名.....类名.方法名(参数列表)
      访问修饰符可以省略
      返回值可以使用通配符表示任意返回值
      包名可以使用通配符表示任意包，但是有几级包，就需要写几个*
      包名可以使用..表示当前包及其子包
      类名和方法名都可以用*
      参数列表可以直接写数据类型
      基本类型写名称，引用类型写包名.类名
      可以使用通配符匹配所有参数，但必须有参数
      使用..既可以匹配所有有参数的，也可以匹配无参的
      全通配写法
      * *..*.*(..)-->

    <!--配置Logger类-->
    <bean id="logger" class="com.xcq.utils.Logger"></bean>

    <!--配置aop-->
    <aop:config>
        <!--配置切入点表达式 id属性用于指定表达式的唯一标识 expression指定表达式内容
            此标签写在aspect中，只能当前切面使用，它也可以写在aop:aspect外面，且必须要写在aspect之前，此时就变成了所有切面可用-->
        <aop:pointcut id="pt1" expression="execution(* com.xcq.service.impl.*.*(..))"/>
        <!--配置切面-->
        <aop:aspect id="logAdvice" ref="logger">
            <!--配置通知的类型并且建立通知方法和切入点方法的关联-->
           <!-- <aop:before method="beforePrintLog"
                        pointcut-ref="pt1"></aop:before>

            <aop:after-returning method="afterPrintLog"
                        pointcut-ref="pt1"></aop:after-returning>
            <aop:after-throwing method="afterThrowingPrintLog"
                        pointcut-ref="pt1"></aop:after-throwing>
            <aop:after method="afterFinalPrintLog"
                        pointcut-ref="pt1"></aop:after>-->
            <!--配置环绕标签-->
            <aop:around method="aroundPrintLog" pointcut-ref="pt1"></aop:around>
        </aop:aspect>
    </aop:config>
</beans>
```
## 注解aop
```java{.line-numbers}
package com.xcq.utils;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.*;
import org.springframework.stereotype.Component;

/**
 * 用于记录日志的工具类
 */
@Component("logger")
@Aspect//表示当前类是一个切面类
public class Logger {

    @Pointcut("execution(* com.xcq.service.impl.*.*(..))")
    private void pt1(){

    }
    /**
     * 前置通知
     */
    @Before("pt1()")
    public void beforePrintLog(){
        System.out.println("Logger类中的beforePrintLog开始打印日志");
    }

    /**
     * 后置通知
     */
    @AfterReturning("pt1()")
    public void afterPrintLog(){
        System.out.println("Logger类中的afterPrintLog开始打印日志");
    }

    /**
     * 异常通知
     */
    @AfterThrowing("pt1()")
    public void afterThrowingPrintLog() {
        System.out.println("Logger类中的afterThrowingPrintLog开始打印日志");

    }

    /**
     * 最终通知
     */
    @After("pt1()")
    public void afterFinalPrintLog(){
        System.out.println("Logger类中的afterFinalPrintLog开始打印日志");

    }

    /**
     * 环绕通知
     */
    @Around("pt1()")
    public Object aroundPrintLog(ProceedingJoinPoint proceedingJoinPoint){
        Object retVal = null;
        try {
            System.out.println("Logger类中的aroundPrintLog开始打印日志...前置");
            Object[] args = proceedingJoinPoint.getArgs();//得到方法执行所需的参数
            retVal = proceedingJoinPoint.proceed(args);//明确调用业务层方法
            System.out.println("Logger类中的aroundPrintLog开始打印日志...后置");
            return retVal;
        }catch (Throwable t){
            System.out.println("Logger类中的aroundPrintLog开始打印日志...异常");
        }finally {
            System.out.println("Logger类中的aroundPrintLog开始打印日志...最终");
        }
        return null;
    }
}
```
```xml{.line-numbers}
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop
        http://www.springframework.org/schema/aop/spring-aop.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">

    <!--配置spring创建容器时要扫描的包-->
    <context:component-scan base-package="com.xcq"></context:component-scan>

    <!--配置spring开启注释aop的支持-->
    <aop:aspectj-autoproxy></aop:aspectj-autoproxy>
</beans>
```
# spring中的JdbcTemplate
## 作用
用于和数据库交互，实现对表的crud操作
**基本使用**
```java{.line-numbers}
DriverManagerDataSource ds = new DriverManagerDataSource();
        ds.setDriverClassName("com.mysql.jdbc.Driver");
        ds.setUrl("jdbc:mysql://localhost:3306/eesy");
        ds.setUsername("root");
        ds.setPassword("1234");

        //1.创建JdbcTemplate对象
        JdbcTemplate jt = new JdbcTemplate();
        //给jt设置数据源
        jt.setDataSource(ds);
        //2.执行操作
        jt.execute("insert into account(name,money)values('ccc',1000)");
```
**CRUD操作**
```java{.line-numbers}
package com.itheima.jdbctemplate;

import com.itheima.domain.Account;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.core.RowMapper;

import java.sql.ResultSet;
import java.sql.SQLException;

/**
 * JdbcTemplate的CRUD操作
 */
public class JdbcTemplateDemo3 {

    public static void main(String[] args) {
        //1.获取容器
        ApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml");
        //2.获取对象
        JdbcTemplate jt = ac.getBean("jdbcTemplate",JdbcTemplate.class);
        //3.执行操作
        //保存
//        jt.update("insert into account(name,money)values(?,?)","eee",3333f);
        //更新
//        jt.update("update account set name=?,money=? where id=?","test",4567,7);
        //删除
//        jt.update("delete from account where id=?",8);
        //查询所有
//        List<Account> accounts = jt.query("select * from account where money > ?",new AccountRowMapper(),1000f);
//        List<Account> accounts = jt.query("select * from account where money > ?",new BeanPropertyRowMapper<Account>(Account.class),1000f);
//        for(Account account : accounts){
//            System.out.println(account);
//        }
        //查询一个
//        List<Account> accounts = jt.query("select * from account where id = ?",new BeanPropertyRowMapper<Account>(Account.class),1);
//        System.out.println(accounts.isEmpty()?"没有内容":accounts.get(0));

        //查询返回一行一列（使用聚合函数，但不加group by子句）
        Long count = jt.queryForObject("select count(*) from account where money > ?",Long.class,1000f);
        System.out.println(count);


    }
}

/**
 * 定义Account的封装策略
 */
class AccountRowMapper implements RowMapper<Account>{
    /**
     * 把结果集中的数据封装到Account中，然后由spring把每个Account加到集合中
     * @param rs
     * @param rowNum
     * @return
     * @throws SQLException
     */
    @Override
    public Account mapRow(ResultSet rs, int rowNum) throws SQLException {
        Account account = new Account();
        account.setId(rs.getInt("id"));
        account.setName(rs.getString("name"));
        account.setMoney(rs.getFloat("money"));
        return account;
    }
}
```
# spring中的事务控制
## 基于xml 
```xml{.line-numbers}
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/tx
        http://www.springframework.org/schema/tx/spring-tx.xsd
        http://www.springframework.org/schema/aop
        http://www.springframework.org/schema/aop/spring-aop.xsd">

    <!-- 配置业务层-->
    <bean id="accountService" class="com.itheima.service.impl.AccountServiceImpl">
        <property name="accountDao" ref="accountDao"></property>
    </bean>

    <!-- 配置账户的持久层-->
    <bean id="accountDao" class="com.itheima.dao.impl.AccountDaoImpl">
        <property name="dataSource" ref="dataSource"></property>
    </bean>


    <!-- 配置数据源-->
    <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="driverClassName" value="com.mysql.jdbc.Driver"></property>
        <property name="url" value="jdbc:mysql://localhost:3306/eesy"></property>
        <property name="username" value="root"></property>
        <property name="password" value="1234"></property>
    </bean>

    <!-- spring中基于XML的声明式事务控制配置步骤
        1、配置事务管理器
        2、配置事务的通知
                此时我们需要导入事务的约束 tx名称空间和约束，同时也需要aop的
                使用tx:advice标签配置事务通知
                    属性：
                        id：给事务通知起一个唯一标识
                        transaction-manager：给事务通知提供一个事务管理器引用
        3、配置AOP中的通用切入点表达式
        4、建立事务通知和切入点表达式的对应关系
        5、配置事务的属性
               是在事务的通知tx:advice标签的内部

     -->
    <!-- 配置事务管理器 -->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"></property>
    </bean>

    <!-- 配置事务的通知-->
    <tx:advice id="txAdvice" transaction-manager="transactionManager">
        <!-- 配置事务的属性
                isolation：用于指定事务的隔离级别。默认值是DEFAULT，表示使用数据库的默认隔离级别。
                propagation：用于指定事务的传播行为。默认值是REQUIRED，表示一定会有事务，增删改的选择。查询方法可以选择SUPPORTS。
                read-only：用于指定事务是否只读。只有查询方法才能设置为true。默认值是false，表示读写。
                timeout：用于指定事务的超时时间，默认值是-1，表示永不超时。如果指定了数值，以秒为单位。
                rollback-for：用于指定一个异常，当产生该异常时，事务回滚，产生其他异常时，事务不回滚。没有默认值。表示任何异常都回滚。
                no-rollback-for：用于指定一个异常，当产生该异常时，事务不回滚，产生其他异常时事务回滚。没有默认值。表示任何异常都回滚。
        -->
        <tx:attributes>
            <tx:method name="*" propagation="REQUIRED" read-only="false"/>
            <tx:method name="find*" propagation="SUPPORTS" read-only="true"></tx:method>
        </tx:attributes>
    </tx:advice>

    <!-- 配置aop-->
    <aop:config>
        <!-- 配置切入点表达式-->
        <aop:pointcut id="pt1" expression="execution(* com.itheima.service.impl.*.*(..))"></aop:pointcut>
        <!--建立切入点表达式和事务通知的对应关系 -->
        <aop:advisor advice-ref="txAdvice" pointcut-ref="pt1"></aop:advisor>
    </aop:config>
</beans>         
```
## 基于注解
```xml{.line-numbers}
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/tx
        http://www.springframework.org/schema/tx/spring-tx.xsd
        http://www.springframework.org/schema/aop
        http://www.springframework.org/schema/aop/spring-aop.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">

    <!-- 配置spring创建容器时要扫描的包-->
    <context:component-scan base-package="com.itheima"></context:component-scan>

    <!-- 配置JdbcTemplate-->
    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
        <property name="dataSource" ref="dataSource"></property>
    </bean>



    <!-- 配置数据源-->
    <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="driverClassName" value="com.mysql.jdbc.Driver"></property>
        <property name="url" value="jdbc:mysql://localhost:3306/eesy"></property>
        <property name="username" value="root"></property>
        <property name="password" value="1234"></property>
    </bean>

    <!-- spring中基于注解 的声明式事务控制配置步骤
        1、配置事务管理器
        2、开启spring对注解事务的支持
        3、在需要事务支持的地方使用@Transactional注解


     -->
    <!-- 配置事务管理器 -->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"></property>
    </bean>



    <!-- 开启spring对注解事务的支持-->
    <tx:annotation-driven transaction-manager="transactionManager"></tx:annotation-driven>

</beans>
```
```java{.line-numbers}
package com.itheima.service.impl;

import com.itheima.dao.IAccountDao;
import com.itheima.domain.Account;
import com.itheima.service.IAccountService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Propagation;
import org.springframework.transaction.annotation.Transactional;

/**
 * 账户的业务层实现类
 *
 * 事务控制应该都是在业务层
 */
@Service("accountService")
@Transactional(propagation= Propagation.SUPPORTS,readOnly=true)//只读型事务的配置
public class AccountServiceImpl implements IAccountService{

    @Autowired
    private IAccountDao accountDao;


    @Override
    public Account findAccountById(Integer accountId) {
        return accountDao.findAccountById(accountId);

    }


    //需要的是读写型事务配置
    @Transactional(propagation= Propagation.REQUIRED,readOnly=false)
    @Override
    public void transfer(String sourceName, String targetName, Float money) {
        System.out.println("transfer....");
            //2.1根据名称查询转出账户
            Account source = accountDao.findAccountByName(sourceName);
            //2.2根据名称查询转入账户
            Account target = accountDao.findAccountByName(targetName);
            //2.3转出账户减钱
            source.setMoney(source.getMoney()-money);
            //2.4转入账户加钱
            target.setMoney(target.getMoney()+money);
            //2.5更新转出账户
            accountDao.updateAccount(source);

            int i=1/0;

            //2.6更新转入账户
            accountDao.updateAccount(target);
    }
}
```