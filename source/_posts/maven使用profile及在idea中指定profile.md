---
title: maven使用profile及在idea中指定profile
---
## 背景
在开发过程中经常遇到使用中间件，在本地调试时pom.xml中的scope需要指定为compile，而提交到平台时需要需要指定为provided。每次debug和package时需要来回切换，一点不优雅。

## 建立profile

```xml
    <profiles>
        <profile>
            <id>dev</id>
            <activation>
                <activeByDefault>true</activeByDefault>
            </activation>
            <properties>
                <mscope>provided</mscope>
            </properties>
        </profile>
        <profile>
            <id>local</id>
            <properties>
                <mscope>compile</mscope>
            </properties>
        </profile>
    </profiles>
```

## 扫描目录
如果profile中有参数值需要同步到配置文件，需要配置过滤目录。

```xml
    <build>
        <resources>
            <resource>
                <directory>${project.basedir}/src/main/resources</directory>
                <filtering>true</filtering>
            </resource>
        </resources>
    </build>
```

## 指定profile
打包时mvn命令后面添加-Pdev指定使用dev profile文件。

```shell
mvn package -Pdev
```

## idea中指定profile
idea中要指定local profile：
> Maven Projects -> Profiles -> 勾选local

这样无论本地debug还是run都默认使用local profile

