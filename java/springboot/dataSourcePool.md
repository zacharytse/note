[TOC]
# 基本选型
常用的数据库连接池有两种，Druid和HikariCP
#HikariCP
## 单数据源
依赖导入
```xml{.line-numbers}
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
<dependency> <!-- 本示例，我们使用 MySQL -->
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>5.1.48</version>
</dependency>
```
配置文件:
```properties
spring.datasource.url=jdbc:mysql://localhost:3306/spring?serverTimezone=UTC
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.username=root
spring.datasource.password=xie2481
#池中维护的最小空闲连接数，默认为10个
spring.datasource.hikari.minimum-idle=10
#池中维护的最大空闲连接数，默认为10个
spring.datasource.hikari.maximum-pool-size=10
```
测试
```java{.line-numbers}
@SpringBootApplication
public class Application implements CommandLineRunner {

    private Logger logger = LoggerFactory.getLogger(Application.class);

    @Autowired
    private DataSource dataSource;

    public static void main(String[] args) {
        // 启动 Spring Boot 应用
        SpringApplication.run(Application.class, args);
    }

    @Override
    public void run(String... args) {
        try (Connection conn = dataSource.getConnection()) {
            // 这里，可以做点什么
            logger.info("[run][获得连接：{}]", conn);
        } catch (SQLException e) {
            throw new RuntimeException(e);
        }
    }

}
```
## 多数据源
配置文件
```properties{.line-numbers}
#first的数据源配置
spring.datasource.first.url=jdbc:mysql://localhost:3306/spring?serverTimezone=UTC
spring.datasource.first.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.first.username=root
spring.datasource.first.password=xie2481
#池中维护的最小空闲连接数，默认为10个
spring.datasource.first.hikari.minimum-idle=10
#池中维护的最大空闲连接数，默认为10个
spring.datasource.first.hikari.maximum-pool-size=10
#second的数据源配置
spring.datasource.second.url=jdbc:mysql://localhost:3306/mybatisdb?serverTimezone=UTC
spring.datasource.second.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.second.username=root
spring.datasource.second.password=xie2481
#池中维护的最小空闲连接数，默认为10个
spring.datasource.second.hikari.minimum-idle=10
#池中维护的最大空闲连接数，默认为10个
spring.datasource.second.hikari.maximum-pool-size=10
```
配置类
```java{.line-numbers}
package com.xcq.springbootdatasourcepool.config;

import com.zaxxer.hikari.HikariDataSource;
import org.springframework.boot.autoconfigure.jdbc.DataSourceProperties;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import org.springframework.util.StringUtils;

import javax.sql.DataSource;

@Configuration
public class DataSourceConfig {

    /**
     * 创建first数据源的配置对象
     */
    @Primary
    @Bean(name = "firstDataSourceProperties")
    @ConfigurationProperties(prefix = "spring.datasource.first")
    public DataSourceProperties firstDataSourceProperties() {
        return new DataSourceProperties();
    }

    /**
     * 创建first数据源
     */
    @Bean(name = "firstDataSource")
    @ConfigurationProperties(prefix = "spring.datasource.first.hikari")
    public DataSource firstDataSource() {
        //获得DataSourceProperties对象
        DataSourceProperties properties = firstDataSourceProperties();
        //创建HikariDataSource对象
        return createHikariDataSource(properties);
    }

    @Bean(name = "secondDataSourceProperties")
    @ConfigurationProperties(prefix = "spring.datasource.second")
    public DataSourceProperties secondDataSourceProperties() {
        return new DataSourceProperties();
    }

    /**
     * 创建second数据源
     */
    @Bean(name = "secondDataSource")
    @ConfigurationProperties(prefix = "spring.datasource.second.hikari")
    public DataSource secondDataSource() {
        //获得DataSourceProperties对象
        DataSourceProperties properties = firstDataSourceProperties();
        //创建HikariDataSource对象
        return createHikariDataSource(properties);
    }

    private static HikariDataSource createHikariDataSource(DataSourceProperties properties) {
        //创建HikariDataSource都西昂
        HikariDataSource dataSource = properties.initializeDataSourceBuilder()
                .type(HikariDataSource.class).build();
        //设置线程池名
        if(StringUtils.hasText(properties.getName())){
            dataSource.setPoolName(properties.getName());
        }
        return dataSource;
    }
}
```
测试
```java{.line-numbers}
package com.xcq.springbootdatasourcepool;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.context.properties.ConfigurationProperties;

import javax.annotation.Resource;
import javax.sql.DataSource;
import java.sql.Connection;
import java.sql.SQLException;

@SpringBootApplication
public class SpringbootDatasourcePoolApplication implements CommandLineRunner {

    private Logger logger = LoggerFactory.getLogger(getClass());

    @Resource(name = "firstDataSource")
    private DataSource firstDataSource;

    @Resource(name = "secondDataSource")
    private DataSource secondDataSource;

    public static void main(String[] args) {
        SpringApplication.run(SpringbootDatasourcePoolApplication.class, args);
    }


    @Override
    public void run(String... args) throws Exception {
        /*try(Connection conn = dataSource.getConnection()){
            logger.info("[run][获得连接:{}]",conn);
        }catch (SQLException e){
            throw new RuntimeException(e);
        }*/
        try (Connection conn = firstDataSource.getConnection()){
            logger.info("[run][firstDataSource 获得连接: {}]",conn);
        }catch (SQLException e){
            throw new RuntimeException(e);
        }

        try (Connection conn = secondDataSource.getConnection()){
            logger.info("[run][secondDataSource 获得连接: {}]",conn);
        }catch (SQLException e){
            throw new RuntimeException(e);
        }
    }
}
```