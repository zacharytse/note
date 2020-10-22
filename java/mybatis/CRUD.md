[TOC]
#CRUD
##基本的修改和删除
```java{.line-numbers}
//IUserDao.java
package mybatis.dao;

import mybatis.annotation.Select;
import mybatis.domain.User;

import java.util.List;

public interface IUserDao {
    /**
     * 查询所有操作
     * */
    //@Select("select * from user")
    List<User> findAll();

    /**
     * 保存用户
     * @param user
     */
    void saveUser(User user);

    /**
     * 更新用户
     * @param user
     */
    void updateUser(User user);

    /**
     * 根据ID删除user
     * @param userId
     */
    void deleteUser(Integer userId);

    /**
     * 根据id查询用户信息
     * @param userId
     * @return
     */
    User findById(Integer userId);

    /**
     * 根据名称模糊查询用户名称
     * @param userName
     * @return
     */
    List<User> findByName(String userName);

    /**
     * 查询总用户数
     * @return
     */
    int findTotal();
}
```
IUserDao.xml的配置文件如下
```xml{.line-numbers}
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="mybatis.dao.IUserDao">
    <select id="findAll" resultType="mybatis.domain.User">
        select * from user
    </select>
    <!-- 保存用户-->
    <insert id="saveUser" parameterType="mybatis.domain.User">
        insert  into user(username,address,sex,birthday)values(#{username},#{address},#{sex},#{birthday});
    </insert>

    <!--更新用户-->
    <update id="updateUser" parameterType="mybatis.domain.User">
        update user set username=#{username},address=#{address},sex=#{sex},birthday=#{birthday} where id = #{id};
    </update>

    <!--删除用户,#{id}里面的id写什么都无所谓，只是一个占位符-->
    <delete id="deleteUser" parameterType="int">
        delete from user where id = #{id}
    </delete>

    <!--根据id查询用户-->
    <select id="findById" parameterType="int" resultType="mybatis.domain.User">
        select * from user where id= #{id}
    </select>

    <!--根据名称模糊查询-->
    <select id="findByName" parameterType="string" resultType="mybatis.domain.User">
        select * from user where username like #{userName}
    </select>

    <!--获取用户的总记录条数-->
    <select id="findTotal" resultType="int">
        select count(id) from user
    </select>
</mapper>
```
测试代码如下
```java{.line-numbers}

import mybatis.dao.IUserDao;
import mybatis.domain.User;
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import org.junit.After;
import org.junit.Before;
import org.junit.Test;

import java.io.IOException;
import java.io.InputStream;
import java.util.Date;
import java.util.List;

public class MybatisTest {
    private InputStream in;
    private SqlSession sqlSession;
    private IUserDao userDao;
    @Before//用于在测试方法执行之前执行
    public void init() throws IOException {
        //1.读取配置文件
        in = Resources.getResourceAsStream("SqlMapConfig.xml");
        //2.创建SqlSessionFactory工厂
        SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
        SqlSessionFactory factory = builder.build(in);
        //3.使用工厂生产一个SqlSession对象
        sqlSession = factory.openSession();
        //4.使用SqlSession创建Dao接口的代理对象
        userDao = sqlSession.getMapper(IUserDao.class);
    }

    @After//用于在测试方法之前执行
    public void destroy() throws IOException{
        sqlSession.commit();
        sqlSession.close();
        in.close();
    }

    @Test
    public void testFindAll() throws IOException {
        List<User> users = userDao.findAll();
        for(User user : users){
            System.out.println(user);
        }
    }

    /**
     * 测试保存操作
     */
    @Test
    public void testSave() throws IOException {
        User user = new User();
        user.setUsername("mybatis saveuser");
        user.setAddress("beijing");
        user.setSex("男");
        user.setBirthday(new Date());
        userDao.saveUser(user);
    }

    /**
     * 测试更新操作
     */
    @Test
    public void testUpdate(){
        User user = new User();
        user.setId(49);
        user.setUsername("mybatis newuser");
        user.setAddress("beijing");
        user.setSex("女");
        user.setBirthday(new Date());
        userDao.updateUser(user);
    }

    /**
     * 执行删除操作
     */
    @Test
    public void testDelete(){
        userDao.deleteUser(49);
    }

    /**
     * 测试查询一个用户
     */
    @Test
    public void testFindOne(){
        User user = userDao.findById(48);
        System.out.println(user);
    }

    /**
     * 模糊查询
     */
    @Test
    public void testFindByName(){
        List<User> users = userDao.findByName("%王%");
        for(User user : users){
            System.out.println(user);
        }
    }

    /**
     * 测试查询总记录条数
     */
    @Test
    public void testFindTotal(){
        int count = userDao.findTotal();
        System.out.println(count);
    }
}

```
**对于模糊查询**
也可以写成
```xml
 <!--根据名称模糊查询-->
    <select id="findByName" parameterType="string" resultType="mybatis.domain.User">
        select * from user where username like '%${value}%';
    </select>
```
${}中必须要写成value
因为在源码绑定sql时，是直接写死了value这个单词
写成下面
```xml
<select id="findByName" parameterType="string" resultType="mybatis.domain.User">
        select * from user where username like #{userName}
</select>
```
在底层是用PreparedStatement的参数占位符，
而写成
```xml
 <!--根据名称模糊查询-->
    <select id="findByName" parameterType="string" resultType="mybatis.domain.User">
        select * from user where username like '%${value}%';
    </select>
```
在底层是用Statement对象的字符串拼接SQL,所以采用#{userName}写法更好
##在插入时获取到插入对象在数据库中的id
```xml{.line-numbers}
<!-- 保存用户-->
    <insert id="saveUser" parameterType="mybatis.domain.User">
        <!--配置插入操作后，获取插入数据的id-->
        <!--keyProperty对应实体类的属性名,keyColumn对应数据库的列名-->
        <selectKey keyProperty="id" keyColumn="id" resultType="int" order="AFTER">
            select last_insert_id();
        </selectKey>
        insert  into user(username,address,sex,birthday)values(#{username},#{address},#{sex},#{birthday});
    </insert>
```
```java{.line-numbers}
 @Test
    public void testSave() throws IOException {
        User user = new User();
        user.setUsername("mybatis saveuser insertid");
        user.setAddress("beijing");
        user.setSex("男");
        user.setBirthday(new Date());
        System.out.println("before : " + user);
        userDao.saveUser(user);
        System.out.println("after : " + user);
    }
```
#参数
- parameterType(输入类型)
- 传递简单类型
- 传递pojo对象
  使用ognl表达式解析对象字段的值，#{}或者${}括号中的值为pojo属性名称
ognl表达式:Object Graphic Navigation Language(对象图导航语言)。它是通过对象的取值方法来获取数据，在写法上把get省略了。
比如说获取用户的名称:类中的写法：user.getUserName(),ognl表达式写法：user.username。
**mybatis为什么能直接写username,而不用user.xxxx?**
因为在parameterType中已经提供了属性所属的类，所以不需要写对象名
##传递pojo包装对象
```java{.line-numbers}
//IUserDao.java
....
 /**
     * 根据queryVo中的条件查询用户
     * @param vo
     * @return
     */
    List<User> findUserByVo(QueryVo vo);
```
```java{.line-numbers}
package mybatis.domain;

public class QueryVo {
    private User user;

    public User getUser() {
        return user;
    }

    public void setUser(User user) {
        this.user = user;
    }
}
```
```xml
 <!--根据queryVo的条件查询用户-->
    <select id="findUserByVo" parameterType="mybatis.domain.QueryVo" resultType="mybatis.domain.User">
        select * from user where username like #{user.username}
    </select>
```
```java{.line-numbers}
@Test
    public void testFindByVo(){
        QueryVo vo = new QueryVo();
        User user = new User();
        user.setUsername("%王%");
        vo.setUser(user);
        List<User> users = userDao.findUserByVo(vo);
        for(User u : users){
            System.out.println(u);
        }
    }
```
##将查询结果的列名与实体类的属性名进行映射
```xml
<!--配置查询结果的列名和实体类的属性名的对应关系-->
    <resultMap id="userMap" type="mybatis.domain.User">
        <!--主键字段的对应-->
        <id property="userId" column="id"></id>
        <!--对非主键字段的对应-->
        <result property="userName" column="username"></result>
    </resultMap>
    <select id="findAll" resultMap="userMap">
        select * from user
    </select>
```

#mybatis基于传统dao的方式(只需要了解)
```java{.line-numbers}
package mybatis.dao.impl;

import mybatis.dao.IUserDao;
import mybatis.domain.User;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;

import java.util.List;
import java.util.Objects;

public class UserDaoImpl implements IUserDao {
    private SqlSessionFactory factory;

    public UserDaoImpl(SqlSessionFactory factory){
        this.factory = factory;
    }
    @Override
    public List<User> findAll() {
        //获取sqlSession
        SqlSession session = factory.openSession();
        List<User> users = session.selectList("mybatis.dao.IUserDao.findAll");//参数就是获取配置信息的key
        session.close();
        return users;
    }

    @Override
    public void saveUser(User user) {

    }

    @Override
    public void updateUser(User user) {

    }

    @Override
    public void deleteUser(Integer userId) {

    }

    @Override
    public User findById(Integer userId) {
        return null;
    }

    @Override
    public List<User> findByName(String userName) {
        return null;
    }

    @Override
    public int findTotal() {
        return 0;
    }
}
```
```java{.line-numbers}
public class MybatisTest {
    private InputStream in;
    private IUserDao userDao;
    @Before//用于在测试方法执行之前执行
    public void init() throws IOException {
        //1.读取配置文件
        in = Resources.getResourceAsStream("SqlMapConfig.xml");
        //2.创建SqlSessionFactory工厂
        SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
        SqlSessionFactory factory = builder.build(in);
        //3使用工厂对象创建dao对象
        userDao = new UserDaoImpl(factory);
    }

    @After//用于在测试方法之前执行
    public void destroy() throws IOException{
        in.close();
    }

    @Test
    public void testFindAll() throws IOException {
        List<User> users = userDao.findAll();
        for(User user : users){
            System.out.println(user);
        }
    }
}
```
**PreparedStatement对象的执行方法**
- execute:它能执行crud中的任意一种语句。返回值是一个boolean，表示是否有结果集，有结果集是true，否则是truetrue
- executeUpdate:它只能执行cud语句，查询语句无法执行。它的返回值是影响数据库记录的行数
- executeQuery:它只能执行select语句，无法执行增删改。执行结果封装的结果集ResultSet对象
mybatis dao执行过程源码分析
![](https://gitee.com/zacharytse/image/raw/master/img/20201021141819.png)
![](https://gitee.com/zacharytse/image/raw/master/img/20201021141830.png)
![](https://gitee.com/zacharytse/image/raw/master/img/20201021141838.png)
![](https://gitee.com/zacharytse/image/raw/master/img/20201021141848.png)
![](https://gitee.com/zacharytse/image/raw/master/img/20201021141857.png)
#mybatis配置
##properties标签
```xml{.line-numbers}
<configuration>
    <!--配置properties-->
    <properties>
        <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
        <property name="url" value="jdbc:mysql://localhost:3306/mybatisdb?serverTimezone=UTC"/>
        <property name="username" value="root"/>
        <property name="password" value="xie2481"/>
    </properties>
    <!--配置环境-->
    <environments default="mysql">
        <!--配置id为mysql的环境-->
        <environment id="mysql">
            <!--表示事物的类型-->
            <transactionManager type="JDBC"></transactionManager>
            <!--配置数据源(连接池)-->
            <dataSource type="POOLED">
                <!--配置连接数据库的四个基本信息-->
                <property name="driver" value="${driver}"/>
                <property name="url" value="${url}"/>
                <property name="username" value="${username}"/>
                <property name="password" value="${password}"/>
            </dataSource>
        </environment>
    </environments>
```
引入外部配置文件
```
//jdbcConfig.properties
jdbc.driver=com.mysql.cj.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/mybatisdb?serverTimezone=UTC
jdbc.username=root
jdbc.password=xie2481
```
```xml{.line-numbers}
<properties resource="jdbcConfig.properties">
    </properties>
    <!--配置环境-->
    <environments default="mysql">
        <!--配置id为mysql的环境-->
        <environment id="mysql">
            <!--表示事物的类型-->
            <transactionManager type="JDBC"></transactionManager>
            <!--配置数据源(连接池)-->
            <dataSource type="POOLED">
                <!--配置连接数据库的四个基本信息-->
                <property name="driver" value="${jdbc.driver}"/>
                <property name="url" value="${jdbc.url}"/>
                <property name="username" value="${jdbc.username}"/>
                <property name="password" value="${jdbc.password}"/>
            </dataSource>
        </environment>
    </environments>
```
url标签
```xml{.line-numbers}
 <properties url="file:///F:\javalearn\learn-spring\src\main\resources\jdbcConfig.properties">
    </properties>
    <!--配置环境-->
    <environments default="mysql">
        <!--配置id为mysql的环境-->
        <environment id="mysql">
            <!--表示事物的类型-->
            <transactionManager type="JDBC"></transactionManager>
            <!--配置数据源(连接池)-->
            <dataSource type="POOLED">
                <!--配置连接数据库的四个基本信息-->
                <property name="driver" value="${jdbc.driver}"/>
                <property name="url" value="${jdbc.url}"/>
                <property name="username" value="${jdbc.username}"/>
                <property name="password" value="${jdbc.password}"/>
            </dataSource>
        </environment>
    </environments>
```
##typeAliases标签
```xml{.line-numbers}
<configuration>
    <!--配置properties
    可以在标签内部配置连接数据库的信息，也可以通过属性引用外部配置文件信息
    resource属性，用于指定配置文件的位置，是按照类路径的写法，并且必须存在于类路径下-->
    <!--使用typeAliases配置别名,它只能配置domain中类的别名-->
    <typeAliases>
        <!--typeAlias用于配置别名，type属性指定实体类全限定类名，alias属性指定别名，当
        指定了别名就不再区分大小写-->
        <typeAlias type="mybatis.domain.User" alias="user"></typeAlias>
    </typeAliases>
    <!--配置环境-->
    <environments default="mysql">
        <!--配置id为mysql的环境-->
        <environment id="mysql">
            <!--表示事物的类型-->
            <transactionManager type="JDBC"></transactionManager>
            <!--配置数据源(连接池)-->
            <dataSource type="POOLED">
                <!--配置连接数据库的四个基本信息-->
                <property name="driver" value="${jdbc.driver}"/>
                <property name="url" value="${jdbc.url}"/>
                <property name="username" value="${jdbc.username}"/>
                <property name="password" value="${jdbc.password}"/>
            </dataSource>
        </environment>
    </environments>
```
```xml{.line-numbers}
 <!--用于指定要配置别名的包，当指定之后，该包下的实体类都会注册别名，并且类名就是别名，不再
    区分大小写-->
    <typeAliases>
        <package name="mybatis.domain"/>
    </typeAliases>
```
##mappers标签
```xml{.line-numbers}
<mappers>
        <!--package是用于指定dao接口所在的包，当指定完成之后，就不需要再写mapper以及resource
        或者class-->
        <package name="mybatis.dao"/>
       <!-- <mapper resource="mybatis/dao/IUserDao.xml"/>-->
        <!--<mapper class="mybatis.dao.IUserDao"/>-->
    </mappers>
```
#mybatis连接池以及事务控制
**连接池**
在实际开发中都会使用连接池，因为它可以减少我们获取连接所消耗的时间。
连接池就是用于存储连接的容器
##mybatis连接池
提供了3种方式的配置，配置的位置:
主配置文件SqlMapConfig.xml的datasource标签，type属性就是表示采用何种连接池方式。

**type属性取值**
- POOLED
采用javax.sql.DataSource规范中的连接池，mybatis中有针对规范的实现
- UNPOOLED
  采用传统的获取连接的方式，虽然也实现了javax.sql.DataSource接口，但是并没有使用池的思想
- JNDI
采用服务器提供的JNDI技术实现，来获取DataSource对象，不同的服务器所能拿到的DataSoruce是不一样的。要注意的是，如果不是web或者maven的war工程，是不能使用的。实际开发中使用的是tomcat服务器，采用的连接池就是dbcp连接池6  
##连接池使用及分析
- POOLED 
  是从池中获取一个连接来用
  ![](https://gitee.com/zacharytse/image/raw/master/img/mybatis_pooled的过程.png)
- UNPOOLED
  每次创建一个新的连接来用
##事务控制的分析
###什么是事务
###事务的四大特性ACID
###不考虑隔离会产生的3个问题
###解决方法：四种隔离级别
#mybatis基于xml配置的动态sql语句使用(会用)
##mappers配置文件中的几个标签
###\<if\>
```xml{.line-numbers}
<select id="findUserByCondition" resultType="user" parameterType="user">
        select * from user where 1=1
        <if test="username != null">
           and username = #{username}
        </if>
</select>
```
###\<where\>
```xml{.line-numbers}
<select id="findUserByCondition" resultType="user" parameterType="user">
        select * from user
        <where>
            <if test="username != null">
                and username = #{username}
            </if>
            <if test="sex != null">
                and sex = #{sex}
            </if>
        </where>
</select>
```
###\<foreach\>
```xml{.line-numbers}
 <select id="findUserInIds" resultType="user" parameterType="QueryVo">
        select * from user
        <where>
            <if test="ids != null and ids.size()>0">
                <foreach collection="ids" open="and id in (" close=")" item="id" separator=",">
                    #{id}
                </foreach>
            </if>
        </where>
    </select>
```
###\<sql\>
```xml{.line-numbers}
<!--了解的内容，抽取重复的sql语句-->
    <sql id="defaultUser">
        select * from user;
    </sql>
    <select id="findAll" resultType="user">
        <include refid="defaultUser"></include>
    </select>
```
#mybatis多表操作(重要)
##表之间的关系
- 一对多
  用户和订单之间是一对多
- 多对一
  订单和用户之间是一对多
- 一对一
  一个身份证只能属于一个人，一个人也只能有一个身份证
- 多对多
  一个学生可以被多个老师教过，一个老师可以教多个学生

**特例**
如果拿出**每一个**订单，他都只能属于一个用户，所以mybatis把多对一看成了一对一

##实例
用户和账户
一个用户可以有多个账户
一个账户只能属于一个用户(多个账户也可以属于同一个用户)

**步骤**
1. 建立两张表，一张用户表，一张账户表
   让用户表和账户表之间具备一对多的关系，需要使用外键在账户表中添加
2. 建立两个实体类，用户实体类和账户实体类
   让用户和账户的实体类能体现出一对多的关系
3. 建立两个配置文件
   用户的配置文件
   账户的配置文件
4. 实现配置：
   当我们查询用户时，可以同时得到用户下所包含的账户信息
   当我们查询账户时，可以同时得到账户的所属用户信息
首先建立IAccountDao的接口
```java{.line-numbers}
package com.xcq.dao;

import com.xcq.domain.Account;
import com.xcq.domain.AccountUser;

import java.util.List;

public interface IAccountDao {

    /**
     * 查询所有账户,同时还要获取到当前账户的所属用户信息
     * @return
     */
    List<Account> findAll();

    /**
     * 查询所有账户，并且带有用户名称和地址信息
     * @return
     */
    List<AccountUser> findAllAccount();
}
```
IUserDao的接口如下
```java{.line-numbers}
package com.xcq.dao;
import com.xcq.domain.User;

import java.util.List;

public class IUserDao {
    /**
     * 查询所有用户
     * @return
     */
    List<User> findAll() {
        return null;
    }

    /**
     * 根据id查询用户信息
     * @param userId
     * @return
     */
    User findById(Integer userId) {
        return null;
    }
}
```
两个接口对应的配置文件如下
```xml{.line-numbers}
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.xcq.dao.IAccountDao">
    <select id="findAll" resultType="account">
        select * from account;
    </select>

    <!--查询所有账户同时包含用户名和地址信息-->
    <select id="findAllAccount" resultType="accountuser">
        select a.*,u.username,u.address from account a,user u where u.id = a.uid;
    </select>
</mapper>
```
```xml{.line-numbers}
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.xcq.dao.IUserDao">
    <select id="findAll" resultType="user">
        select * from user;
    </select>

    <!--根据id查询用户-->
    <select id="findById" parameterType="int" resultType="user">
        select * from user where id= #{id}
    </select>

</mapper>
```
测试程序如下
```java{.line-numbers}
 @Test
    public void testFindAllAccountUser(){
        List<AccountUser> aus = accountDao.findAllAccount();
        for(AccountUser accountUser : aus){
            System.out.println(accountUser);
        }
    }
```
**使用mybatis定义的一对多查询**
```xml{.line-numbers}
<resultMap id="accountUserMap" type="account">
        <id property="id" column="aid"></id>
        <result property="uid" column="uid"></result>
        <result property="money" column="money"></result>
        <!--一对一的关系映射，配置封装user的内容-->
        <!--association中的column是对应的外键-->
        <association property="user" column="uid" javaType="user">
            <id property="id" column="id"></id>
            <result column="username" property="username"></result>
            <result column="address" property="address"></result>
            <result column="sex" property="sex"></result>
            <result column="birthday" property="birthday"></result>
        </association>
    </resultMap>

    <select id="findAll" resultMap="accountUserMap">
        select u.*,a.id as aid,a.uid,a.money from account a,user u where u.id = a.uid
    </select>
```
**mybatis实现一对多的查询操作**
```java{.line-numbers}
package com.xcq.domain;

import java.io.Serializable;

public class Account implements Serializable {
    private Integer id;
    private Integer uid;
    private Double money;

    //从表实体应该包含一个主表实体的对象引用
    private User user;

    public User getUser() {
        return user;
    }

    public void setUser(User user) {
        this.user = user;
    }

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public Integer getUid() {
        return uid;
    }

    public void setUid(Integer uid) {
        this.uid = uid;
    }

    public Double getMoney() {
        return money;
    }

    public void setMoney(Double money) {
        this.money = money;
    }

    @Override
    public String toString() {
        return "Account{" +
                "id=" + id +
                ", uid=" + uid +
                ", money=" + money +
                '}';
    }
}
```
```xml{.line-numbers}
 <resultMap id="userAccountMap" type="user">
        <id property="id" column="id"></id>
        <result property="username" column="username"></result>
        <result property="address" column="address"></result>
        <result property="sex" column="sex"></result>
        <result property="birthday" column="birthday"></result>
        <!--配置user对象中accounts集合的映射 ofType是集合中元素的类型-->
        <collection property="accounts" ofType="account">
            <id column="aid" property="id"></id>
            <result column="uid" property="uid"></result>
            <result column="money" property="money"></result>
        </collection>
    </resultMap>
    <select id="findAll" resultMap="userAccountMap">
        select * from user u left outer join account a on u.id = a.uid;
    </select>
```
**实例2**
用户和角色
用户可以有多个角色
一个角色也可以有多个用户
**步骤**
1. 建立两张表，一张用户表，一张角色表
   让用户表和角色表具有多对多的关系，需要使用中间表，中间表中包含各自的主键，在中间表中是外键
2. 建立两个实体类，用户实体类和角色实体类
   让用户和角色的实体类能体现出多对多的关系
   各自包含对方一个集合引用
3. 建立两个配置文件
   用户的配置文件
   角色的配置文件
4. 实现配置：
   当我们查询用户时，可以同时得到用户下所包含的角色信息
   当我们查询角色时，可以同时得到角色的所属用户信息
**查询角色获取所属用户信息的实现**
```java{.line-numbers}
package com.xcq.domain;

import java.io.Serializable;
import java.util.List;

public class Role implements Serializable {
    private Integer roleId;
    private String roleName;
    private String roleDesc;

    //多对多的关系映射,一个角色可以赋予多个用户
    private List<User> users;

    public List<User> getUsers() {
        return users;
    }

    public void setUsers(List<User> users) {
        this.users = users;
    }

    public Integer getRoleId() {
        return roleId;
    }

    public void setRoleId(Integer roleId) {
        this.roleId = roleId;
    }

    public String getRoleName() {
        return roleName;
    }

    public void setRoleName(String roleName) {
        this.roleName = roleName;
    }

    public String getRoleDesc() {
        return roleDesc;
    }

    public void setRoleDesc(String roleDesc) {
        this.roleDesc = roleDesc;
    }

    @Override
    public String toString() {
        return "Role{" +
                "roleId=" + roleId +
                ", roleName='" + roleName + '\'' +
                ", roleDesc='" + roleDesc + '\'' +
                '}';
    }
}
```
```xml{.line-numbers}
<!--定义role表的resultMap-->
    <resultMap id="roleMap" type="role">
        <id property="roleId" column="rid"></id>
        <result property="roleName" column="role_name"></result>
        <result property="roleDesc" column="role_desc"></result>
        <collection property="users" ofType="user">
            <id property="id" column="id"></id>
            <result column="username" property="username"></result>
            <result column="address" property="address"></result>
            <result column="sex" property="sex"></result>
            <result column="birthday" property="birthday"></result>
        </collection>
    </resultMap>
    <!--查询所有-->
    <select id="findAll" resultMap="roleMap">
        select u.*,r.id as rid,r.role_name,r.role_desc from role r
        left outer join user_role ur on r.id = ur.rid
        left outer join user u on u.id = ur.uid
    </select>
```
**实现一个用户查询多个角色**
```java{.line-numbers}
package com.xcq.domain;

import java.io.Serializable;
import java.sql.Date;
import java.util.List;

public class User implements Serializable {
    private Integer id;
    private String username;
    private String address;
    private String sex;
    private Date birthday;

    //多对多的关系映射，一个用户可以具备多个角色
    private List<Role> roles;

    public List<Role> getRoles() {
        return roles;
    }

    public void setRoles(List<Role> roles) {
        this.roles = roles;
    }

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getAddress() {
        return address;
    }

    public void setAddress(String address) {
        this.address = address;
    }

    public String getSex() {
        return sex;
    }

    public void setSex(String sex) {
        this.sex = sex;
    }

    public Date getBirthday() {
        return birthday;
    }

    public void setBirthday(Date birthday) {
        this.birthday = birthday;
    }

    @Override
    public String toString() {
        return "User{" +
                "id=" + id +
                ", username='" + username + '\'' +
                ", address='" + address + '\'' +
                ", sex='" + sex + '\'' +
                ", birthday=" + birthday +
                '}';
    }
}
```
```xml{.line-numbers}
<resultMap id="userMap" type="user">
        <id property="id" column="id"></id>
        <result property="username" column="username"></result>
        <result property="address" column="address"></result>
        <result property="sex" column="sex"></result>
        <result property="birthday" column="birthday"></result>
        <!--配置角色集合的映射-->
        <collection property="roles" ofType="role">
            <id property="roleId" column="rid"></id>
            <result property="roleName" column="role_name"></result>
            <result property="roleDesc" column="role_desc"></result>
        </collection>
    </resultMap>
    <select id="findAll" resultMap="userMap">
        select u.*,r.id as rid,r.role_name,r.role_desc from user u
        left outer join user_role ur on u.id = ur.uid
        left outer join role r on r.id = ur.rid
    </select>
```
#JDNI
Java Naming And Directory Interface。目的是模仿windows系统中的注册表
jndi是一个map结构，value是对应的对象，key存的是路径(directory)加名称(naming)。其中directory是固定的，name是自己指定的。要存放的对象也是可以指定的，是通过配置文件指定的
从p60开始看
