---
title: mybatis + spring + druid demo
---


## 使用mybatis-spring步骤
* 建立dataSource bean
* 建立sqlSessionFactory bean
* 建立mapper bean
* 配置自动扫描mapper

[mybatis-spring官方文档](http://www.mybatis.org/spring/zh/)

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
app-config.xml

```xml

<?xml version="1.0" encoding="UTF-8"?>
<!--suppress SpringFacetInspection -->
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans  http://www.springframework.org/schema/beans/spring-beans.xsd">
    <description>Spring公共配置文件</description>

    <!--ads dataSource-->
    <bean id="testDataSource" class="com.alibaba.druid.pool.DruidDataSource">
        <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
        <property name="url" value="${db.url}"/>
        <property name="username" value="${db.user}"/>
        <property name="password" value="${db.password}"/>
        <property name="maxActive" value="10"/>
        <property name="minIdle" value="0"/>
        <property name="maxWait" value="5000"/>
    </bean>

    <!--mybatis sqlSessionFactory-->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="testDataSource"/>
        <!-- 加载mybatis的配置文件 -->
        <property name="configLocation" value="mybatis/mybatis-config.xml"/>
    </bean>

    <!--userMapper-->
    <bean id="userMapper" class="org.mybatis.spring.mapper.MapperFactoryBean">
        <property name="mapperInterface" value="com.zart.fun.mapper.UserMapper"/>
        <property name="sqlSessionFactory" ref="sqlSessionFactory"/>
    </bean>

    <!-- 自动扫描mapper-->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="basePackage" value="com.zart.fun.mapper"/>
        <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"/>
    </bean>

</beans>

```

---
mybatis-config.xml

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

</configuration>
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
        ApplicationContext applicationContext = new ClassPathXmlApplicationContext("classpath:spring/app-config.xml");
        UserMapper mapper = (UserMapper)applicationContext.getBean("userMapper");

        System.out.println(mapper.getUser().toString());
    }
}

```


---
结果

```shell
[UserEntity{id=1, name='LiLei', age=15}, UserEntity{id=2, name='HanMeimei', age=15}]
```
