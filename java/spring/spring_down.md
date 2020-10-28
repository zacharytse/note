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
## AOP相关术语
# 基于xml和注解的AOP配置
spring  day3 08