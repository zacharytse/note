[TOC]
#延迟加载
问题：在一对多种，当我们有一个用户，它有100个账户。
> 在查询用户的时候，要不要把关联的账户查出来?

在查询用户时，用户下的账户信息应该是什么时候使用，什么时候查询。
> 在查询账户的时候，要不要把关联的用户查出来？

在查询账户时，账户的所属用户信息应该是随着账户查询时一起查询出来
##延迟加载
在真正使用数据时，才发起查询，不用的时候不查询，也叫按需加载(懒加载)
> 在使用时，将mybatis的版本设置为3.5.1
##立即加载
不管用不用，只要一调用方法，马上发起查询

**在对应的四种表关系中，一对多，多对一，一对一，多对多**

一对多,多对多：通常情况下我们都是采用延迟加载

多对一，一对一:通常情况下我们都是采用立即加载

###一对一实现延迟加载
在主配置文件中开启延迟加载的选项
```xml
<settings>
        <!--开启mybatis支持延迟加载-->
        <setting name="lazyLoadingEnabled" value="true"/>
        <setting name="aggressiveLazyLoading" value="false"/>
    </settings>
```
配置需要延迟加载的对象
```xml
<resultMap id="accountUserMap" type="account">
        <id property="id" column="id"></id>
        <result property="uid" column="uid"></result>
        <result property="money" column="money"></result>
        <!-- 一对一的关系映射：配置封装user的内容-->
        <!--select属性指定的内容，查询用户的唯一标识-->
        <!--column属性指定的内容：用户根据id查询时，所需要的参数的值-->
        <association property="user" column="uid" javaType="user" select="com.itheima.dao.IUserDao.findById">
        </association>
    </resultMap>

    <!-- 查询所有 -->
    <select id="findAll" resultMap="accountUserMap">
        select * from account
    </select>
```
###多对一实现延迟加载
```xml
<resultMap id="userAccountMap" type="user">
        <id property="id" column="id"></id>
        <result property="username" column="username"></result>
        <result property="address" column="address"></result>
        <result property="sex" column="sex"></result>
        <result property="birthday" column="birthday"></result>
        <!-- 配置user对象中accounts集合的映射 -->
        <collection property="accounts" ofType="account" select="com.itheima.dao.IAccountDao.findAccountByUid"
        column="id">
        </collection>
    </resultMap>

    <!-- 查询所有 -->
    <select id="findAll" resultMap="userAccountMap">
        select * from user
    </select>
```
```xml
<select id="findAccountByUid" resultType="account" parameterType="int">
        select * from account where uid = #{uid}
    </select>
```
#缓存
##什么是缓存
存在于内存中的临时数据
##为什么使用缓存
减少和数据库的交互次数，提高执行效率
##什么样的数据能使用缓存，什么样的数据不能使用
> 适用于缓存的数据

- 经常查询且不经常改变
- 数据的正确与否对最终结果影响不大
  
> 不适用于缓存的数据

- 经常改变的数据
- 数据的正确与否对最终结果影响很大，例如商品的库存，银行的汇率等等
##一级缓存和二级缓存
###一级缓存
mybatis中SqlSession对象的缓存，当我们执行查询之后，查询的结果会同时存入到SqlSession为我们提供的一块区域中。该区域结构是一个map。当我们再次查询同样的数据，mybatis会先去SqlSession中查看是否有，有的话直接拿出来用
当SqlSession对象消失时，mybatis一级缓存也就消失了
###二级缓存
mybatis中SqlSessionFactory对象的缓存，由SqlSessionFactory对象创建的SqlSession共享其缓存
并且二级缓存中存放的是**数据不是对象**

使用步骤
1. 让mybatis框架支持二级缓存
   ```xml
   <settings>
        <setting name="cachedEnabled" value="true"/>
    </settings>
   ```
2. 让当前的映射文件支持二级缓存
   ```xml
    <!--开启user支持二级缓存-->
    <cache/>
   ```
3. 让当前的操作支持二级缓存
   ```xml
   <select id="findById" parameterType="INT" resultType="user" useCache="true">
        select * from user where id = #{uid}
    </select>
   ```
**当调用SqlSession的修改，添加，删除，commit(),close()等方法时，会清空一级缓存**
#注解开发
```java{.line-numbers}
package com.xcq.dao;

import com.xcq.domain.User;
import org.apache.ibatis.annotations.Delete;
import org.apache.ibatis.annotations.Insert;
import org.apache.ibatis.annotations.Select;
import org.apache.ibatis.annotations.Update;

import java.util.List;

/**
 * mybatis中针对CRUD一共有四个注解
 * @Select @Insert @Update @Delete
 */
public interface IUserDao {

    /**
     * 查询所有用户
     * @return
     */
    @Select("select * from user")
    List<User> findAll();

    /**
     * 保存用户
     * @param user
     */
    @Insert("insert into user(username,address,sex,birthday)values(#{username},#{address},#{sex},#{birthday})")
    void saveUser(User user);

    /**
     * 更新用户
     * @param user
     */
    @Update("update user set username=#{username},sex=#{sex},birthday=#{birthday},address=#{address} where id = #{id}")
    void updateUser(User user);

    /**
     * 删除用户
     * @param userId
     */
    @Delete("delete from user where id = #{id}")
    void deleteUser(Integer userId);

    /**
     * 根据id查询用户
     * @param userId
     * @return
     */
    @Select("select * from user where id = #{id}")
    User findById(Integer userId);

    /**
     * 根据用户名称模糊查询
     * @param username
     * @return
     */
    //@Select("select * from user where username like #{username}")
    @Select("select * from user where username like '%${value}%'")
    List<User> findUserByName(String username);

    /**
     * 查询总用户数量
     * @return
     */
    @Select("select count(*) from user")
    int findTotal();
}
```
```xml{.line-numbers}
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <!--引入外部配置文件-->
    <properties resource="jdbcConfig.properties"></properties>
    <!--配置别名-->
    <typeAliases>
        <package name="com.xcq.domain"/>
    </typeAliases>
    <!--配置环境-->
    <environments default="mysql">
        <environment id="mysql">
            <transactionManager type="jdbc"></transactionManager>
            <dataSource type="POOLED">
                <property name="driver" value="${jdbc.driver}"/>
                <property name="url" value="${jdbc.url}"/>
                <property name="username" value="${jdbc.username}"/>
                <property name="password" value="${jdbc.password}"/>
            </dataSource>
        </environment>
    </environments>
    <!--指定带有注解的dao接口所在的位置-->
    <mappers>
        <package name="com.xcq.dao"/>
    </mappers>
</configuration>
```
通过注解起别名，这里懒的给dao改名字了
```java{.line-numbers}
 @Select("select * from user where username like '%${value}%'")
    @Results(id = "userMap",value = {
            @Result(id = true,column = "id",property = "id"),
            @Result(column = "username",property = "username"),
            @Result(column = "address",property = "address"),
            @Result(column = "sex",property = "sex"),
            @Result(column = "birthday",property = "birthday")
    })
    List<User> findUserByName(String username);

    /**
     * 查询总用户数量
     * @return
     */
    @Select("select count(*) from user")
    @ResultMap(value = {"userMap"})
    int findTotal();
```
##一对一配置
```java{.line-numbers}
@Select("select * from account")
    @Results(id = "accountMap",value = {
            @Result(id = true,column = "id",property = "id"),
            @Result(id = true,column = "uid",property = "uid"),
            @Result(id = true,column = "money",property = "money"),
            @Result(property = "user",column = "uid",one = @One(select="com.xcq.dao.IUserDao.findById",fetchType= FetchType.EAGER))
    })
    List<Account> findAll();
```
##一对多配置
```java{.line-numbers}
@Select("select * from user")
    @Results(id = "userMap",value = {
            @Result(id = true,column = "id",property = "id"),
            @Result(id = true,column = "username",property = "username"),
            @Result(id = true,column = "address",property = "address"),
            @Result(id = true,column = "sex",property = "sex"),
            @Result(id = true,column = "birthday",property = "birthday"),
            @Result(property = "accounts",column = "id",
                    many = @Many(select = "com.xcq.dao.IAccountDao.findAccountByUid",
                            fetchType = FetchType.LAZY))
    })
    List<User> findAll();
```
**注解开启二级缓存**
```java{.line-numbers}
@CacheNamespace(blocking = true)
public interface IUserDao {
}
```