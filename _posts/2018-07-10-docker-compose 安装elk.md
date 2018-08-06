---
layout: post
title: 'docker-compose 安装elk'
date: 2018-07-10
author: yunfei
tags: docker elk
---

### 1、安装elk
### 1.1 Docker compose文件
```yaml
version:"3.1"

services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:6.3.0
    container_name: elasticsearch
    restart: always
    environment:
      - "http.host=0.0.0.0"
      - "transport.host=127.0.0.1"
      - "bootstrap.memory_lock=true"
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      - "xpack.security.enabled: false"
      - "xpack.monitoring.enabled: false"
      - "xpack.graph.enabled: false"
      - "xpack.watcher.enabled: false"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    ports:
      - "9200:9200"
      - "9300:9300"
    user: "elasticsearch"
    volumes:
      - /docker/elasticsearch/logs:/usr/share/elasticsearch/logs
      -/docker/elasticsearch/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml
      -/docker/elasticsearch/data:/usr/share/elasticsearch/data

  kibana:
    image: docker.elastic.co/kibana/kibana:6.3.0
    container_name: kibana
    restart: always
    links:
      - elasticsearch
    environment:
      - "ELASTICSEARCH_URL=http://elasticsearch:9200"
    ports:
      - "5601:5601"
    volumes:
      - /docker/kibana/data:/docker/kibana/data
      - /docker/kibana/kibana.yml:/usr/share/kibana/config/kibana.yml

  logstash:
    image: docker.elastic.co/logstash/logstash:6.3.0
    container_name: logstash
    restart: always
    depends_on:
      - elasticsearch
    links:
      - elasticsearch
    environment:
      - "ELASTICSEARCH_URL=http://elasticsearch:9200"  
    ports:
      - "5044:5044"
    volumes:
      -/docker/logstash/logstash.conf:/usr/share/logstash/pipeline/logstash.conf
      -/docker/logstash/pipeline:/usr/share/logstash/pipeline
      - /docker/logstash/temp:/temp
      -/docker/logstash/logstash.yml:/usr/share/logstash/config/logstash.yml
```

### 1.2 Elasticsearch.yml文件
放到如下目录：docker compose里映射的目录
/docker/elasticsearch/config/elasticsearch.yml
```yaml
cluster.name: "docker-cluster"
network.host: "0.0.0.0"
discovery.zen.minimum_master_nodes: 1
xpack.security.enabled: false
```

### 1.3 logstash.yml文件
放到如下目录：docker compose里映射的目录
/docker/logstash/logstash.yml
```yaml
http.host: "0.0.0.0"
path.config: /usr/share/logstash/pipeline
xpack.monitoring.enabled: true
xpack.monitoring.elasticsearch.url: http://192.168.2.210:9200
xpack.monitoring.elasticsearch.username: elastic
xpack.monitoring.elasticsearch.password: changeme
pipeline.workers: 10
pipeline.output.workers: 10
pipeline.batch.size: 3000
pipeline.batch.delay: 100
```

### 1.4 kibana.yml文件
```yaml
server.host: "0.0.0.0"
xpack.security.enabled: true
elasticsearch.username: "elastic"
elasticsearch.password: "changeme" 
xpack.security.encryptionKey: "something_at_least_32_characters"
xpack.security.sessionTimeout: 600000
xpack.monitoring.ui.container.elasticsearch.enabled: true
```

### 1.5 logstash.conf文件
/docker/logstash/logstash.conf
/docker/logstash/pipeline
```text
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
  #stdout {codec => rubydebug }
}
``` 

### 2、Kibana 创建elasticsearch索引
### 2.1 访问http://ip:5601
选择dev tools>执行以下语句
```text
PUT /test
{
 "mappings": {
  "doc": {
   "properties": {
    "speaker":{"type": "keyword"},
    "play_name":{"type": "keyword"},
    "line_id":{"type": "integer"},
    "speech_number":{"type": "integer"}
   }
  }
 }
}
```
![image png](/assets/img/putTest.png)

### 2.2 查看索引
执行：GET /_cat/indices?v

![image png](/assets/img/index.png)

### 3、 Elasticsearch api
1、curl -XGETlocalhost:9200/.monitoring-kibana-6-2018.05.31?pretty
![image png](/assets/img/eapi1.png)

2、查看索引：
http://192.xx.xx.xx:9200/_cat/indices?v

3、删除索引：
curl -XDELETE http://localhost:9200/my_index?pretty
 
4、导入模板到elasticsearch：
curl -XPUT -H 'Content-Type:application/json' http://localhost:9200/_template/filebeat-7.0.0-alpha1 -d@filebeat-index-template.json


### 4、错误     
### 4.1 logstash 报ruby错误
Does it try to require a relative path?That's been removed in Ruby 1.9.
uri:classloader:/META-INF/jruby.home/lib/ruby/stdlib/rubygems/core_ext/kernel_require.rb:59:in`require':
It seems your ruby installation is missingpsych (for YAML output).
To eliminate this warning, please installlibyaml and reinstall your ruby.
[ERROR] 2018-05-31 02:09:06.151 [main]Logstash - java.lang.IllegalStateException:org.jruby.exceptions.RaiseException: (GemspecError) There was a LoadError whileloading logstash-core.gemspec:
load error: psych --java.lang.RuntimeException: BUG: we can not copy embedded jar to temp directory


解决方式：
![image png](/assets/img/handle.png)

文件目录要有写入权限，不能为只读，例如/usr/share/logstash/pipeline/logstash.conf:ro就是只读的

 

### 4.2 elasticsearch没有权限
![image png](/assets/img/e1.png)

设置宿主目录的权限

![image png](/assets/img/e2.png)

![image png](/assets/img/e3.png)

### 4.3 elasticsearch
max virtual memory areas vm.max_map_count[65530] is too low, increase to at least [262144]

![image png](/assets/img/e4.png)

sysctl -w vm.max_map_count=262144

![image png](/assets/img/e5.png)


### 4.4 Kibana6.x.x—启动后的一些警告信息记录以及解决方法
1、发现的第一个警告信息
server  log   [06:55:25.594] [warning][reporting] Generating a random key for xpack.reporting.encryptionKey. 　　　　　　　　　　　　　　　　 To prevent pending reports from failing on restart, please set xpack.reporting.encryptionKey in kibana.yml
根据提示，在配置文件kibana.yml中添加
【xpack.reporting.encryptionKey】属性：
xpack.reporting.encryptionKey: "a_random_string"
官方文档：https://www.elastic.co/guide/en/kibana/current/reporting-settings-kb.html


2、发现的第二个警告信息
server   log   [06:55:25.686] [warning][security] Generating a random key for xpack.security.encryptionKey. 　　　　　　　　　　　　　　　　　　To prevent sessions from being invalidated on restart, please set xpack.security.encryptionKey in kibana.yml
根据提示，在配置文件kibana.yml中添加
【xpack.security.encryptionKey】属性：
xpack.security.encryptionKey: "something_at_least_32_characters"
官方文档：https://www.elastic.co/guide/en/kibana/6.x/using-kibana-with-security.html


