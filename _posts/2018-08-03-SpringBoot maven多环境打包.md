---
layout: post
title: 'SpringBoot maven多环境打包'
date: 2018-08-03
author: yunfei
tags: SpringBoot Maven
---

### 1、maven pom.xml配置

####（1）添加profires
````xml
<profiles>
   <!--开发环境-->
   <profile>
      <id>dev</id>
      <properties>
         <build.profile.id>dev</build.profile.id>
      </properties>
      <activation>
         <activeByDefault>true</activeByDefault>
      </activation>
   </profile>
   <!--测试环境-->
   <profile>
      <id>test</id>
      <properties>
         <build.profile.id>test</build.profile.id>
      </properties>
   </profile>
   <!--生产环境-->
   <profile>
      <id>produce</id>
      <properties>
         <build.profile.id>produce</build.profile.id>
      </properties>
   </profile>
</profiles>
````

####（2）在build下添加resource

````xml
<resources>
   <resource>
      <directory>${project.basedir}/src/main/resources</directory>
      <filtering>true</filtering>
      <includes>
         <include>application.yml</include>
      </includes>
   </resource>
   <resource>
      <directory>${project.basedir}/src/main/resources</directory>
      <includes>
         <include>**/*</include>
      </includes>
   </resource>
</resources>
````

####（3）在plugins里添加配置，允许使用${}获取maven变量值

````xml
<plugin>
   <artifactId>maven-resources-plugin</artifactId>
   <configuration>
      <encoding>utf-8</encoding>
      <useDefaultDelimiters>true</useDefaultDelimiters>
   </configuration>
</plugin>
````

### 2、springboot 配置文件

${build.profile.id} 获取maven变量值
```yaml
spring:
  profiles:
    active: ${build.profile.id}
```


### 3、打包命令

正式环境打包
mvn clean package -DskipTests–P produce

开发环境打包
mvn clean package -DskipTests–P dev

测试环境打包
mvn clean package -DskipTests–P test
