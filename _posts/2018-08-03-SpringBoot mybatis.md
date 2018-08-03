---
layout: post
title: 'SpringBoot mybatis'
date: 2018-08-02
author: yunfei
tags: SpringBoot mybatis
---

### 1、generator配置
````xml
 <build>
      <plugins>
         <!--MG的插件 -->
         <plugin>
            <groupId>org.mybatis.generator</groupId>
            <artifactId>mybatis-generator-maven-plugin</artifactId>
            <version>1.3.5</version>
            <dependencies>
                <!-- https://mvnrepository.com/artifact/mysql/mysql-connector-java 配置这个依赖主要是为了等下在配置MG的时候可以不用配置classPathEntry这样的一个属性，避免代码的耦合度太高 -->
                <dependency>
                   <groupId>mysql</groupId>
                  <artifactId>mysql-connector-java</artifactId>
                   <version>5.1.43</version>
                </dependency>
 
                <dependency>
                   <groupId>org.mybatis.generator</groupId>
                   <artifactId>mybatis-generator-core</artifactId>
                   <version>1.3.5</version>
                </dependency>
            </dependencies>
            <configuration>
                <verbose>true</verbose>
                <overwrite>true</overwrite>
                <!-- 自动生成的配置 -->
                <configurationFile>
                   src/main/resources/config/mybatis-generator.xml</configurationFile>
            </configuration>
         </plugin>
 
         <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
         </plugin>
      </plugins>
   </build>
````
### 2、新建generatorConfig.xml 文件

放在src/main/resources/config目录下
````xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
<generatorConfiguration>
   <properties resource="application-dev.properties" />
   <!-- <context id="Mysql" targetRuntime="MyBatis3"> -->
   <context id="Mysql" targetRuntime="tk.mybatis.mapper.generator.TkMyBatis3Impl"
      defaultModelType="flat">
 
      <!-- 生成的Java文件的编码 -->
      <property name="javaFileEncoding" value="UTF-8" />
      <!-- 格式化java代码 -->
      <property name="javaFormatter"
         value="org.mybatis.generator.api.dom.DefaultJavaFormatter" />
      <!-- 格式化XML代码 -->
      <property name="xmlFormatter"
         value="org.mybatis.generator.api.dom.DefaultXmlFormatter" />
 
      <plugin type="tk.mybatis.mapper.generator.MapperPlugin">
         <property name="mappers" value="com.comba.config.CombaMapper" />
      </plugin>
      <commentGenerator>
         <!-- 是否去除自动生成的注释 true：是 ： false:否 -->
         <property name="suppressAllComments" value="true" />
         <!-- 是否生成注释代时间戳 -->
         <property name="suppressDate" value="true" />
      </commentGenerator>
      <!--数据库链接地址账号密码 -->
      <jdbcConnection driverClass="${spring.datasource.driver-class-name}"
         connectionURL="${spring.datasource.url}" userId="${spring.datasource.username}"
         password="xxx">
      </jdbcConnection>
      <javaTypeResolver>
         <property name="forceBigDecimals" value="false" />
      </javaTypeResolver>
      <!--生成Model类存放位置 -->
      <javaModelGenerator targetPackage="com.demo.entity"
         targetProject="src/main/java">
         <property name="enableSubPackages" value="true" />
         <!-- 是否针对string类型的字段在set的时候进行trim调用 -->
         <property name="trimStrings" value="true" />
      </javaModelGenerator>
      <!--生成映射文件存放位置 -->
      <sqlMapGenerator targetPackage="mapper" targetProject="src/main/resources">
         <property name="enableSubPackages" value="true" />
      </sqlMapGenerator>
      <!--生成Dao类存放位置 -->
      <!-- 客户端代码，生成易于使用的针对Model对象和XML配置文件 的代码 type="ANNOTATEDMAPPER",生成Java Model 
         和基于注解的Mapper对象 type="MIXEDMAPPER",生成基于注解的Java Model 和相应的Mapper对象 type="XMLMAPPER",生成SQLMap 
         XML文件和独立的Mapper接口 -->
      <javaClientGenerator type="XMLMAPPER"
         targetPackage="com.comba.dao" targetProject="src/main/java">
         <property name="enableSubPackages" value="true" />
      </javaClientGenerator>
      <!--生成对应表及类名 domainObjectName="plan" -->
<!--      <table tableName="master" domainObjectName="Master" mapperName="{0}Dao"
         enableCountByExample="false" enableUpdateByExample="false"
         enableDeleteByExample="false" enableSelectByExample="false"
         selectByExampleQueryId="false">
       <generatedKey column="id" sqlStatement="Mysql" /> 
      </table>-->
      <table tableName="agv_gis_config" enableCountByExample="false" mapperName="{0}Dao"
         enableUpdateByExample="false" enableDeleteByExample="false" 
         enableSelectByExample="false" selectByExampleQueryId="false">        
         <ignoreColumn column="createtime" />
         <ignoreColumn column="modifytime" />
      </table>
   </context>
</generatorConfiguration>
````
### 3、MyEclipse项目右键-RUN AS-MAVEN BUILD..
输入 mybatis-generator:generate 


### 4、集成通用mapper

http://git.oschina.net/free/Mapper
https://gitee.com/free/Mapper
````xml
      <!--通用mapper -->
      <dependency>
         <groupId>tk.mybatis</groupId>
         <artifactId>mapper-spring-boot-starter</artifactId>
         <version>1.1.4</version>
      </dependency>
````

#### （1）接口
````java
package com.demo.config;
 
import tk.mybatis.mapper.common.Mapper;
import tk.mybatis.mapper.common.MySqlMapper;
 
/**
 *
 *@Project demo
 *@Package com.demo.config
 *@ClassName DemoMapper.java
 *@Description 被继承的Mapper，一般业务Mapper继承它
 *@JDK
 *@date  2017年9月5日
 *@Author wangzengfeng
 *@Version
 */
 
public interface DemoMapper<T> extends Mapper<T>, MySqlMapper<T> {
    //TODO
    //FIXME 特别注意，该接口不能被扫描到，否则会出错
}
````
#### （2）配置文件
````yaml
#mybatis
 
#指定bean所在包
mybatis.type-aliases-package=com.comba.entity
#指定映射文件
mybatis.mapperLocations=classpath:mapper/*.xml
#mapper
#mappers 多个接口时逗号隔开
mapper.mappers=com.comba.config.CombaMapper
mapper.not-empty=false
mapper.identity=MYSQL
#pagehelper
pagehelper.helperDialect=mysql
pagehelper.reasonable=true
pagehelper.supportMethodsArguments=true
pagehelper.params=count=countSql
````

<font color="#FF4500"> 说明：
通用 Mapper 支持 Mybatis-3.2.4 及以上版本
不是表中字段的属性必须加@Transient注解</font>

 
### 5、mybatis 插入返回id
可以在id上加注解
@Id
@GeneratedValue(strategy = GenerationType.IDENTITY, generator = "SELECT LAST_INSERT_ID()")
private Long id;

也可以用mybatis封装好的方法：
insertUseGeneratedKeys

### 6、update 返回id
````xml
<update id="updateOrder" parameterType="com.demo.entity.ProductOrder">
    <selectKey keyProperty='id' resultType='Long' order='BEFORE'>
        select
        (select id from product_order where
        product_order_no=#{productOrderNo})id
        from DUAL
    </selectKey>
    update product_order set product_no=#{productNo},product_name=#{productName},produce_date=#{produceDate},
    plan_amount=#{planAmount},start_time=#{startTime} where product_order_no=#{productOrderNo}
</update>
````