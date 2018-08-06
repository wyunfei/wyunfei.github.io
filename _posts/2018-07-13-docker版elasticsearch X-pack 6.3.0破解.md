---
layout: post
title: 'docker版elasticsearch X-pack 6.3.0破解'
date: 2018-07-13
author: yunfei
tags: docker
---

### 1、X-pack 6.3.0破解    
### 1.1 复制 x-pack-core-6.3.0.jar
从elasticsearch docker容器里复制x-pack-core-6.3.0.jar到宿主机
```text
docker cp
2e4a9082e64f:/usr/share/elasticsearch/modules/x-pack/x-pack-core/x-pack-core-6.3.0.jar/docker/elasticsearch/
```

说明：
Elasticsearch容器ID：2e4a9082e64f
x-pack-core-6.3.0.jar容器里位置：
/usr/share/elasticsearch/modules/x-pack/x-pack-core/x-pack-core-6.3.0.jar
宿主机目录：/docker/elasticsearch/
![image png](/assets/img/docker1.png)
 

### 1.2 luyten反编译x-pack-core-6.3.0.jar
用luyten反编译保存为java文件，找到
org.elasticsearch.license.LicenseVerifier.class
org.elasticsearch.xpack.core.XPackBuild.class

luyten项目地址:https://github.com/deathmarine/Luyten
将反编译后的java 代码复制到自己的IDE中，按照同样的包名创建pack
我们不需要编译整个项目，只需要编译这两个文件，所以要把依赖添加到classpath中。
依赖也与之前有所变化，之前只需要x-pack 包本身，现在需要引入 elasticsearch 6.3.0 中 lib 目录下的jar包 以及 x-pack-core-6.3.0.jar 本身

#### 1、修改LicenseVerifier
LicenseVerifier 中有两个静态方法，这就是验证授权文件是否有效的方法，我们把它修改为全部返回true.
![image png](/assets/img/license.png)

#### 2、修改XPackBuild
XPackBuild 中最后一个静态代码块中 try的部分全部删除，这部分会验证jar包是否被修改.
![image png](/assets/img/XPackBuild.png)


### 1.3 把重新编译后的文件添加到x-pack-core-6.3.0.jar
右键解压x-pack-core-6.3.0.jar，然后分别替换
org.elasticsearch.license.LicenseVerifier.class 
org.elasticsearch.xpack.core.XPackBuild.class
![image png](/assets/img/xpack-core.png)

替换后，重新压缩x-pack-core-6.3.0.jar
![image png](/assets/img/xpack-zip.png)
 

### 1.4 替换原来的x-pack-core-6.3.0.jar
复制宿主机的x-pack-core-6.3.0.jar文件到elasticsearch容器里，并重启elasticserach、kinaba
```text
docker cp
/docker/elasticsearch/x-pack-core-6.3.0.jar  2e4a9082e64f:/usr/share/elasticsearch/modules/x-pack/x-pack-core/
```

### 1.5 导入授权文件
#### 1、先从官网申请basic授权文件
https://license.elastic.co/registration
![image png](/assets/img/license-basic.png)


#### 2、授权文件修改
```json
{
   "uid": "6fb96d6b-938c-45ff-9ce7-6b53b39cd7dd",
   "type": "platinum", # 修改授权为白金版本
   "issue_date_in_millis": 1530489600000,
   "expiry_date_in_millis": 2855980923000, #修改到期时间为2060-07-02
   "max_nodes": 100,  # 修改最大节点数
   "issued_to": "xxxx",
   "issuer": "Web Form",
   "signature":"AAAAAwAAAA3PP60wKNtAvRmuCGdSAAABmC9ZN0hjZDBGYnVyRXpCOW5Bb3FjZDAxOWpSbTVoMVZwUzRxVk1PSmkxaktJRVl5MUYvUWh3bHZVUTllbXNPbzBUemtnbWpBbmlWRmRZb25KNFlBR2x0TXc2K2p1Y1VtMG1UQU9TRGZVSGRwaEJGUjE3bXd3LzRqZ05iLzRteWFNekdxRGpIYlFwYkJiNUs0U1hTVlJKNVlXekMrSlVUdFIvV0FNeWdOYnlESDc3MWhlY3hSQmdKSjJ2ZTcvYlBFOHhPQlV3ZHdDQ0tHcG5uOElCaDJ4K1hob29xSG85N0kvTWV3THhlQk9NL01V",
   "start_date_in_millis": 1530489600000
} 
```

时间戳、时间转换
https://tool.lu/timestamp
![image png](/assets/img/timesmap.png)


#### 3、导入授权文件
方式一：通过kibana界面导入
![image png](/assets/img/kibana-license.png)

选择授权文件上传：
![image png](/assets/img/license-upload.png)

上传成功后：
![image png](/assets/img/upload-success.png)

方式二：通过API接口上传
```text
curl -u elastic:elastic -XPUT 'http://es-ip:port/_xpack/license' -H "Content-Type: application/json" -d @/tmp/license.json
```

### 1.6 参考链接
License查看：https://www.elastic.co/subscriptions
破解教程：
https://www.jianshu.com/p/55b5c5d3a89c
http://blog.51cto.com/billy98/2131989



