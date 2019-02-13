---
title: mybatis + durid (不使用spring)
---


## 使用mybatis + durid步骤
* 配置mybatis配置文件
* 建立DruidDataSourceFactory类
* 建立SqlSessionFactory单例
* 使用SqlSession查询数据

[mybatis官方文档](http://www.mybatis.org/mybatis-3/zh/index.html)

## 相关代码

pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>


    <groupId>com.zart.fun</groupId>
    <artifactId>mybatis-spring-demo</artifactId>
    <version>1.0.0-SNAPSHOT</version>


    <dependencies>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.31</version>
        </dependency>
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.4.6</version>
        </dependency>
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis-spring</artifactId>
            <version>1.3.2</version>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>1.1.12</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>5.0.2.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-tx</artifactId>
            <version>5.0.2.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jdbc</artifactId>
            <version>5.0.2.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-lang3</artifactId>
            <version>3.1</version>
        </dependency>
    </dependencies>

    <build>
        <resources>
            <!--保证该目录下*.xml文件打包时不被清掉-->
            <resource>
                <directory>src/main/java</directory>
                <includes>
                    <include>
                        **/*.xml
                    </include>
                </includes>
            </resource>
        </resources>
    </build>

</project>
```
---
mybatis-all-config.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <settings>
        <!--全局地开启或关闭配置文件中的所有映射器已经配置的任何缓存-->
        <setting name="cacheEnabled" value="false"/>
        <!--延迟加载的全局开关-->
        <setting name="lazyLoadingEnabled" value="true"/>
        <!--是否允许单一语句返回多结果集-->
        <setting name="multipleResultSetsEnabled" value="true"/>
        <!--使用列标签代替列名-->
        <setting name="useColumnLabel" value="true"/>
        <!--允许 JDBC 支持自动生成主键，需要驱动兼容-->
        <setting name="useGeneratedKeys" value="false"/>
        <!--指定 MyBatis 应如何自动映射列到字段或属性：NONE, PARTIAL, FULL-->
        <setting name="autoMappingBehavior" value="PARTIAL"/>
        <!--指定发现自动映射目标未知列（或者未知属性类型）的行为：NONE, WARNING, FAILING-->
        <setting name="autoMappingUnknownColumnBehavior" value="WARNING"/>
        <!--配置默认的执行器：SIMPLE REUSE BATCH-->
        <setting name="defaultExecutorType" value="SIMPLE"/>
        <!--设置超时时间，它决定驱动等待数据库响应的秒数-->
        <setting name="defaultStatementTimeout" value="3600"/>
        <!--为驱动的结果集获取数量（fetchSize）设置一个提示值-->
        <setting name="defaultFetchSize" value="10000"/>
        <!--允许在嵌套语句中使用分页（RowBounds）。如果允许使用则设置为false-->
        <setting name="safeRowBoundsEnabled" value="false"/>
        <!--是否开启自动驼峰命名规则（camel case）映射-->
        <setting name="mapUnderscoreToCamelCase" value="false"/>
        <!--MyBatis 利用本地缓存机制（Local Cache）防止循环引用（circular references）和加速重复嵌套查询-->
        <!--SESSION(default)：这种情况下会缓存一个会话中执行的所有查询-->
        <!--STATEMENT：本地会话仅用在语句执行上，对相同 SqlSession 的不同调用将不会共享数据-->
        <setting name="localCacheScope" value="STATEMENT"/>
        <!--当没有为参数提供特定的 JDBC 类型时，为空值指定 JDBC 类型-->
        <setting name="jdbcTypeForNull" value="OTHER"/>
        <!--指定哪个对象的方法触发一次延迟加载-->
        <setting name="lazyLoadTriggerMethods" value="equals,clone,hashCode,toString"/>
    </settings>

    <typeAliases>
        <typeAlias type="com.zart.fun.utils.DruidDataSourceFactory"
                   alias="DRUID" />
    </typeAliases>

    <environments default="test">
        <environment id="test">
            <transactionManager type="JDBC"/>
            <dataSource type="DRUID">
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="${db.url}"/>
                <property name="username" value="${db.user}"/>
                <property name="password" value="${db.password}"/>
                <property name="maxActive" value="10"/>
                <property name="minIdle" value="0"/>
                <property name="maxWait" value="5000"/>
            </dataSource>
        </environment>
    </environments>

    <mappers>
        <mapper resource="com/zart/fun/mapper/UserMapper.xml"/>
    </mappers>

</configuration>
```
---
DruidDataSourceFactory.java

```java
package com.zart.fun.utils;

import java.sql.SQLException;
import java.util.Properties;

import javax.sql.DataSource;

import com.alibaba.druid.pool.DruidDataSource;

import org.apache.commons.lang3.StringUtils;
import org.apache.ibatis.datasource.DataSourceFactory;

/**
 * DruidDataSourceFactory
 *
 * @author zart
 * @date 2018/11/27
 */
public class DruidDataSourceFactory implements DataSourceFactory {
    private Properties props;

    @Override
    public DataSource getDataSource() {
        DruidDataSource dds = new DruidDataSource();
        dds.setDriverClassName(this.props.getProperty("driver"));
        dds.setUrl(this.props.getProperty("url"));
        dds.setUsername(this.props.getProperty("username"));
        dds.setPassword(this.props.getProperty("password"));
        //最大连接池数量
        String maxActive = this.props.getProperty("maxActive");
        if (StringUtils.isNotBlank(maxActive)) {
            int max = Integer.parseInt(maxActive);
            if (max > 0) {
                dds.setMaxActive(max);
            }
        }
        //最小连接池数量
        String minIdle = this.props.getProperty("minIdle");
        if (StringUtils.isNotBlank(minIdle)){
            int min = Integer.parseInt(minIdle);
            if (min >= 0){
                dds.setMinIdle(min);
            }
        }
        //超时时间
        String maxWait = this.props.getProperty("maxWait");
        if (StringUtils.isNotBlank(maxWait)){
            int wait = Integer.parseInt(maxWait);
            if (wait > 0){
                dds.setMaxWait(wait);
            }
        }

        try {
            dds.init();
        } catch (SQLException e) {

        }
        return dds;
    }

    @Override
    public void setProperties(Properties props) {
        this.props = props;
    }
}
```
---
MySqlSessionFactory.java

```java
package com.zart.fun.utils;

import java.io.IOException;

import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;

/**
 * MySqlSessionFactory
 *
 * @author zart
 * @date 2018/11/27
 */
public class MySqlSessionFactory {

    private volatile static org.apache.ibatis.session.SqlSessionFactory sqlSessionFactory;

    /**
     * 获取SpiderSqlSessionFactory单例
     *
     * @return sqlSessionFactory
     */
    public static org.apache.ibatis.session.SqlSessionFactory getFactory() {
        if (sqlSessionFactory == null) {
            synchronized (MySqlSessionFactory.class) {
                if (sqlSessionFactory == null) {
                    sqlSessionFactory = initFactory();
                }
            }
        }
        return sqlSessionFactory;
    }

    /**
     * 初始化SpiderSqlSessionFactory单例
     *
     * @return sqlSessionFactory
     */
    private static org.apache.ibatis.session.SqlSessionFactory initFactory() {
        org.apache.ibatis.session.SqlSessionFactory sqlSessionFactory = null;
        try {
            sqlSessionFactory = new SqlSessionFactoryBuilder().build(
                Resources.getResourceAsReader("mybatis/mybatis-all-config.xml"), "test");
        } catch (IOException ignored) {
        }
        return sqlSessionFactory;
    }
}

```
---
UserMapper Interface

```java
package com.zart.fun.mapper;

import java.util.List;

import com.zart.fun.entity.UserEntity;

/**
 * @author zart
 */
public interface UserMapper {

    /**
     * 获取UserList
     *
     * @return UserList
     */
    List<UserEntity> getUser();
}

```
---
UserMappper.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.zart.fun.mapper.UserMapper">
    <select id="getUser" resultType="com.zart.fun.entity.UserEntity">
        select * from user_test;
    </select>
</mapper>
```

---
UserEntity.java


```java
package com.zart.fun.entity;

/**
 * UserEntity
 *
 * @author zart
 * @date 2018/11/27
 */
public class UserEntity {
    private int id;
    private String name;
    private int age;

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "UserEntity{" +
            "id=" + id +
            ", name='" + name + '\'' +
            ", age=" + age +
            '}';
    }
}

```
---
Main.java


```java
package com.zart.fun;

import com.zart.fun.mapper.UserMapper;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

/**
 * Main
 *
 * @author zart
 * @date 2018/11/27
 */
public class Main {

    public static void main(String[] args){
        // 不使用spring
        SqlSession session = MySqlSessionFactory.getFactory().openSession(true);
        UserMapper mapper1 = session.getMapper(UserMapper.class);
        System.out.println(mapper1.getUser().toString());
}

```
---
结果

```shell
[UserEntity{id=1, name='LiLei', age=15}, UserEntity{id=2, name='HanMeimei', age=15}]
```
