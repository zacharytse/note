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
#mybatis配置
##properties标签
##typeAliases标签
##mappers标签