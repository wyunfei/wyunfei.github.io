---
layout: post
title: 'SpringBoot Swagger在线接口文档'
date: 2018-04-12
author: yunfei
tags: SpringBoot Swagger
---


# Springboot Swagger 在线接口文档

### 1、Swagger配置
#### （1）引入依赖
```xml
  <properties>
      <swagger.version>2.7.0</swagger.version>
   </properties>
      <!-- Swagger -->
      <dependency>
         <groupId>io.springfox</groupId>
         <artifactId>springfox-swagger2</artifactId>
         <version>${swagger.version}</version>
      </dependency>
<!--     <dependency> -->
<!--     <groupId>io.springfox</groupId> -->
<!--     <artifactId>springfox-swagger-ui</artifactId> -->
<!--     <version>${swagger.version}</version> -->
<!--     </dependency> -->
      <!-- 替换swagger ui -->
      <dependency>
         <groupId>com.drore.cloud</groupId>
         <artifactId>swagger-bootstrap-ui</artifactId>
         <version>1.4</version>
    </dependency>
```

#### （2）Swagger配置类
```java
package com.demo.config;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import io.swagger.annotations.ApiOperation;
import springfox.documentation.builders.ApiInfoBuilder;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.service.Contact;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;
 
 
@Configuration
@EnableSwagger2
public class Swagger2 {
         @Bean
         public Docket createRestApi() {
                   return new Docket(DocumentationType.SWAGGER_2)
                                     .apiInfo(apiInfo())
                                     .select()
 
//                                  .apis(RequestHandlerSelectors.basePackage("com.demo.controller")) //把controller包下的所有方法都转为restful api
                           .apis(RequestHandlerSelectors.withMethodAnnotation(ApiOperation.class)) //只把有@ApiOperation注解的方法转为restful api
                                     .paths(PathSelectors.any())
                                     .build();
         }
         private ApiInfo apiInfo() {
                   return new ApiInfoBuilder()
                                     .title("Spring Boot中使用Swagger2构建RESTful APIs")
                                     .description("更多Swagger2说明 https://github.com/swagger-api/swagger-core/wiki/Annotations#apimodel")
                                     .termsOfServiceUrl("http://localhost:8080/")
                                     .contact(new Contact("wzf", "https://swagger.io/", "wangzengfeng@comba.com.cn"))
                                     .version("1.0")
                                     .build();
         }
 
}
```

#### （3）swagger的其他使用方式
```text
参考：https://github.com/SpringForAll/spring-boot-starter-swagger
下载github上面的源码，放到自己工程下面，之后就可以在配置文件里面配置，省去上面的代码配置
```
![image png](/assets/img/swagger.png)
```yaml
swagger:
  title: RESTful APIs
  description: API接口文档
  version: 1.0
  globalOperationParameters[0]:
    name: access_token
    description: token
    parameterType: query
    modelRef: string
    required: true
  docket:
    env:
      base-package: com.demo.web.environment.controller
    system:
      base-package: com.demo.web.system.controller
```

### 2、访问与调试

http://localhost:8080/doc.html
![image png](/assets/img/swaggerApi.png)

调试：
![image png](/assets/img/swaggerTest.png)

### 3、参考
```text
Swagger-Bootstrap-UI是Swagger的前端UI实现,采用jQuery+bootstrap实现,目的是替换Swagger默认的UI实现Swagger-UI,使文档更友好一点：
https://git.oschina.net/xiaoym/swagger-bootstrap-ui
https://oss.sonatype.org/#nexus-search;quick~swagger-bootstrap-ui
```

### 4、Api接口添加全局参数
```java
@Configuration
@EnableSwagger2
public class Swagger2 {
private List<Parameter> parameters (){
    ParameterBuilder aParameterBuilder = new ParameterBuilder();
    aParameterBuilder
            //参数类型支持header, cookie, body, query etc
            .parameterType("query")
            //参数名
            .name("access_token")
            .description("token")
            //指定参数值的类型
            .modelRef(new ModelRef("string"))
            //非必需false，这里是全局配置，然而在登陆的时候是不用验证的
            .required(true).build();
    List<Parameter> aParameters = new ArrayList<Parameter>();
    aParameters.add(aParameterBuilder.build());
    return aParameters;
}
 
@Bean
public Docket createRestApi() {
    return new Docket(DocumentationType.SWAGGER_2)
            .apiInfo(apiInfo())
            .groupName("base")
            .globalOperationParameters(parameters())
            .select()
            //只把有@ApiOperation注解的方法转为restful api
            .apis(RequestHandlerSelectors.withMethodAnnotation(ApiOperation.class))
            .paths(PathSelectors.any())
            .build();
}
}
```
### 5、​​​​​​​问题记录
```text
在通过swagger2在线测试接口时候，如果url中有{id}，需要添加paramType=”path”，否则无法从路径中获得id值
@ApiImplicitParams({
@ApiImplicitParam(name =” id”, value = ”用户ID”, required = true, paramType =”path”, dataType =” Long”),
@ApiImplicitParam(name = ”user”, value = ”用户详细实体user”, required = true, paramType =” body”, dataType = ”User”)
})
要定义paramType， paramType 有五个可选值 ： path, query, body, header, form
```

### 6、生成离线文档

#### （1）创建文件夹

asciidoctor
asciidoctor有maven插件，可以自动把acsiidoc文件转成html和pdf，能自动生成目录，非常方便
先在工程下创建docs/asclidoc这个文件：
![image png](/assets/img/asclidoc.png)
 
文件内容是：
include::{generated}/overview.adoc[]  
include::{generated}/definitions.adoc[]  
include::{generated}/paths.adoc[]  
意思就是引入三个文件

 

#### （2）Maven配置
```xml
   <build>
      <plugins>
         <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
         </plugin>

         <plugin>
            <groupId>org.asciidoctor</groupId>
            <artifactId>asciidoctor-maven-plugin</artifactId>
            <version>1.5.3</version>

            <configuration>
                <sourceDirectory>${project.basedir}/docs/asciidoc</sourceDirectory>
                <sourceDocumentName>index.adoc</sourceDocumentName>
                <attributes>
                   <doctype>book</doctype>
                   <toc>left</toc>
                   <toclevels>3</toclevels>
                   <numbered></numbered>
                   <hardbreaks></hardbreaks>
                   <sectlinks></sectlinks>
                   <sectanchors></sectanchors>
                  <generated>${project.build.directory}/asciidoc</generated>
                </attributes>
            </configuration>

            <!-- Since each execution can only handle one backend, run separate executions
                for each desired output type -->
            <executions>
                <execution>
                   <id>output-html</id>
                   <phase>test</phase>
                   <goals>
                      <goal>process-asciidoc</goal>
                   </goals>

                   <configuration>
                      <backend>html5</backend>
                       <doctype>book</doctype>
                      <!--<outputDirectory>${project.basedir}/docs/asciidoc/html</outputDirectory> -->
                   </configuration>
                </execution>
            </executions>
         </plugin>
      </plugins>
   </build>
```

####（3）运行maven build

运行成功后，在E:\Workspaces\Comba-Api\target\generated-docs文件夹下会生成
![image png](/assets/img/docbuild.png)



