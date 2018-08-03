---
layout: post
title: 'SpringBoot 发送logback日志到elk'
date: 2018-08-03
author: yunfei
tags: SpringBoot elk
---

### 1. 引入依赖

````xml
<dependency>
 <groupId>net.logstash.logback</groupId>
 <artifactId>logstash-logback-encoder</artifactId>
 <version>4.11</version>
</dependency>
````


### 2. 日志配置

在logback.xml文件添加如下配置：
````xml
<appender name="LOGSTASH" 
class="net.logstash.logback.appender.LogstashTcpSocketAppender">
 <destination>192.168.2.210:5044</destination> 
 <!-- encoder必须配置,有多种可选 --> 
 <encoder charset="UTF-8" class="net.logstash.logback.encoder.LogstashEncoder" > 
 <!-- "logName":"comba-community" 的作用是指定创建索引的名字时用，并且在生成的文档中会多了这个字段 -->
  <customFields>{"logName":"comba-community"}</customFields>
 </encoder>
 <filter class="ch.qos.logback.classic.filter.LevelFilter">
 <level>INFO</level>
 </filter>
</appender>

<!-- additivity 避免执行2次 --> 
<logger name="com.comba.controller" level="INFO" 
additivity="false">
 <appender-ref ref="STDOUT" />
 <appender-ref ref="INFO_FILE" />
 <appender-ref ref="ERROR_FILE" />
 <appender-ref ref="DEBUG_FILE" />
 <appender-ref ref="LOGSTASH" />
</logger>

<root level="${root.level}">
 <appender-ref ref="STDOUT" />
 <appender-ref ref="INFO_FILE" />
 <appender-ref ref="ERROR_FILE" />
 <appender-ref ref="DEBUG_FILE" />
 <appender-ref ref="LOGSTASH" />
</root>

````


有多个logstash IP添加如下配置：

````xml
<connectionStrategy> 　　
<roundRobin> 　　　　
<connectionTTL>5 minutes</connectionTTL> 　　
</roundRobin>
</connectionStrategy>
````

这个配置是向logstash输出日志如果有多个logstash IP或端口可以轮询负载各端口

### 3. Logstash配置

````json
input {
 tcp {
 port => 5044
 mode => "server"
 tags => ["tags"]
 codec => json_lines
 }
}
output {
 elasticsearch {
 hosts => "192.168.2.210:9200"
 index => "%{[logName]}-%{+YYYY.MM.dd}"
 user => "elastic"
 password => "changeme"
 }

#控制台输出日志 
#stdout {codec => rubydebug }

}
````

服务器logstash控制输出的日志：

![image.png](https://upload-images.jianshu.io/upload_images/4727108-77fdee4558d70c83.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 4. 参考链接

[https://blog.csdn.net/yy756127197/article/details/78873310](https://blog.csdn.net/yy756127197/article/details/78873310)

[https://blog.csdn.net/u014527058/article/details/70495595](https://blog.csdn.net/u014527058/article/details/70495595)

源码：

[https://github.com/logstash/logstash-logback-encoder](https://github.com/logstash/logstash-logback-encoder)
