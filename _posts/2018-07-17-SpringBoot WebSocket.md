---
layout: post
title: 'SpringBoot WebSocket'
date: 2018-07-17
author: yunfei
tags: SpringBoot WebSocket
---

### 1、添加依赖
````xml
      <dependency>
         <groupId>org.springframework.boot</groupId>
         <artifactId>spring-boot-starter-websocket</artifactId>
      </dependency>
````


### 2、WebSocket配置

````java
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig extends AbstractWebSocketMessageBrokerConfigurer {
 
    @Override
    public void registerStompEndpoints(StompEndpointRegistry stompEndpointRegistry) {
        // 添加服务端点，可以理解为某一服务的唯一key值
        stompEndpointRegistry.addEndpoint("/socket");
        //当浏览器支持sockjs时执行该配置
        stompEndpointRegistry.addEndpoint("/socket").setAllowedOrigins("*").withSockJS();
    }
    
 
    @Override
    public void configureMessageBroker(MessageBrokerRegistry config) {
        // 配置接受订阅消息地址前缀为topic的消息
        config.enableSimpleBroker("/topic");
        // Broker接收消息地址前缀
//        config.setApplicationDestinationPrefixes("/");
 }
}
````
 

3、发送消息

```markdown
   @Autowired
   public SimpMessagingTemplate template;
   template.convertAndSend("/topic/rhythm", push);
```
 

4、前端页面调用
引入sockjs、stomp：
````javascript
<script th:src="@{/js/sockjs.min.js}"></script>
<script th:src="@{/js/stomp.min.js}"></script>
<script th:src="@{/js/jquery-3.1.1.min.js}"></script>
   var s = new SockJS('/socket');
   var stompClient = Stomp.over(s);
   // stompClient.send("/socket", {},JSON.stringify({'push':""})); 
   stompClient.connect({}, function() {
      console.log('notice socket connected!');
      stompClient.subscribe('/topic/rhythm', function(response) {
         var data = JSON.parse(response.body);
         console.log(data);
         initLine('time', "节拍", "", data.legend, data.xdata, data.seriesData);
      });
});
````


