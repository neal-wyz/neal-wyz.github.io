---
layout: post
title: "websocket+spring 实践"
date: 2017-03-26
tags: websocket
---

文章思路：

* WebSocket概览
* spring框架下简单使用javax.websocket包
* 需要注意的问题
* 延伸

## websocket概览
WebSocket协议是HTML5最新规范的一部分，实现了浏览器与服务器全双工(full-duplex)通信。
> WebSocket设计出来的目的就是要取代轮询和    Comet 技术，使客户端浏览器具备像 C/S 架构下桌面系统的实时通讯能力。 浏览器通过 JavaScript 向服务器发出建立 WebSocket 连接的请求，连接建立以后，客户端和服务器端就可以通过 TCP 连接直接交换数据。
> [https://www.ibm.com/developerworks/cn/web/1112_huangxa_websocket/index.html](https://www.ibm.com/developerworks/cn/web/1112_huangxa_websocket/index.html)

WebSocket特点：
* 实现客户端浏览器与服务器全双工实时通信，低延迟。
* 后端技术发展，服务端框架以及各类型语言都集成了WebSocket，最常见的java，tomcat，node.js，python，c#，nginx，php，
* 前端浏览器支持，现在浏览器版本支持情况：Chrome 4+，Firefox 4+，Opera 10+，Safari 5+，IE 10+
* 使用WebSocket协议传输的数据帧头部比http的请求小更加节省流量。
* 建立在TCP基础上的通信。

握手协议：

在使用WebSocket时，客户端浏览器需要给服务端发送一个http/https协议得请求，说明自己要升级为ws/wss协议，即使用WebSocket API来进行后面的通信，服务端经过简单运算，确认身份后，回应一个消息，通过简单加密运算确认身份即抛弃http协议后，在客户端服务端建立ws/wss协议的通道。

WebSocket API：
```javascript
[Constructor(in DOMString url, optional in DOMString protocol)]
interface WebSocket {
  readonly attribute DOMString URL;
  // ready state
  const unsigned short CONNECTING = 0;
  const unsigned short OPEN = 1;
  const unsigned short CLOSED = 2;
  readonly attribute unsigned short readyState;
  readonly attribute unsigned long bufferedAmount;

  // networking
  attribute Function onopen;
  attribute Function onmessage;
  attribute Function onclose;
  boolean send(in DOMString data);
  void close();
};
WebSocket implements EventTarget;
```
定义一个新的WebSocket连接：
```javascript
var myWebSocket = new WebSocket("ws://...");
```
在WebSocket的生命周期中，你可以通过以下三个方法，获取当前WebSocket事件：
```javascript
myWebSocket.onopen = function(evt) { alert("建立连接..."); };
myWebSocket.onmessage = function(evt) { alert("收到消息: " + evt.data); };
myWebSocket.onclose = function(evt) { alert("连接关闭"); };
```
而对于发送数据到服务端，或者想要关闭WebSocket连接，可以使用：
```javascript
myWebSocket.send("消息");
myWebSocket.close();
```

## spring框架下简单使用javax.websocket包

pom.xml
```xml
<dependency>  
    <groupId>javax.websocket</groupId>  
    <artifactId>javax.websocket-api</artifactId>  
    <version>1.0</version>  
    <scope>provided</scope>
</dependency>  
```
建立服务端 WebSocket.java
```java
//使用ServerEndPoint注解表示这是一个WebSocket通信终端，value是地址，encoders提供ws发送的消息编码类。
@ServerEndpoint(value = "/websocket", encoders = {WebSocketEncoder.class})
public class WebSocket {
    
    //使用Map缓存当前连接的session会话
    private static Map<String, Session> allSessions = new HashMap<String, Session>();
    
    //通过spring上下文来获取通过注解注入的javaBean
    private AdvanceOrderService advanceOrderService = (AdvanceOrderService)ContextLoader.getCurrentWebApplicationContext().getBean("advanceOrderService");
	
    private CallService callService = (CallService)ContextLoader.getCurrentWebApplicationContext().getBean("callService");
	
    public WebSocket() {}
    
    /**
     * 向所有客户端群发信息
     */
    public synchronized static void sendAll(String category, String jsonReplyStr) {
        Set<String> keys = allSessions.keySet();
        for (String key : keys) {
            Session session = allSessions.get(key);
            if (session.isOpen()) {
                try {
                    session.getBasicRemote().sendText(jsonReplyStr);
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }
    }
    
    /**
     * 获取客户端发回的消息
     * @param session
     * @param msg
     */
    @OnMessage
    public void onMessage(Session session, String msg) {
    	System.out.println(msg);
    }
    
    /**
     * 建立连接
     */
    @OnOpen
    public void onOpen(Session session, EndpointConfig config) {
        allSessions.put(session.getId(), session);
        System.out.println("建立连接, id=" + session.getId());
    }
    
    /**
     * 关闭连接
     */
    @OnOpen
    public void onOpen(Session session, CloseReason reason) {
        try{
            synchronized (allSessions) {
                allSessions.remove(session.getId());
            }
        }
    }
    
}
```
创建消息编码类 WebSocketEncoder.java
```java
public class WebSocketEncoder implements Encoder.Text<ActionResult>{
    @Override
    public void init(EndpointConfig config) {}
    
    @Override
    public void destroy() {}
    
    @Override
    public String encode(ActionResult actionResult) throws EncodeException {
        return JSONObject.fromObject(actionResult).toString();
    }
}
```
客户端实现 websocket.js
```javascript
$(function() {

    var url = "wss://www.gzswzx.cn/websocket?all";
    var ws = new WebSocket(url);

    ws.onopen = function(e) {
        console.log("与服务器建立链接");
    }

    ws.onmessage = function(e) {
        // 使用js自建方法eval解析字符串为json对象
        var obj = eval('(' + e.data + ')');
        console.log(obj);
    }
    
    ws.onclose = function(e) {
        console.log("与服务器断开链接");
    }
})
```

## 需要注意的问题
* > websocket api在浏览器端的广泛实现似乎只是一个时间问题了，值得注意的是服务器端没有标准的api， 各个实现都有自己的一套api， 并且jcp也没有类似的提案， 所以使用websocket开发服务器端有一定的风险，可能会被锁定在某个平台上或者将来被迫升级。
  > [http://baike.baidu.com/item/WebSocket?sefr=cr](http://baike.baidu.com/item/WebSocket?sefr=cr)
* 在使用注解javax.websocket.server.ServerEndpoint的websocket类中，使用autowired获取不到service需要通过ContextLoader.getCurrentWebApplicationContext().getBean("BeanName")的方法获取service的bean。解决办法：看到一种解释，是因为autowired的注解的属性service是在spring的context中，但是，使用serverendpoint注解的websocket类不归spring管理，两个是在不同的context中，所以获取不到。
* cdn加速目前还不支持WebSocket。


## 延伸
1. 数据通信中，数据在线路上的传送方式可以分为单工通信、半双工通信和全双工通信三种。
* 全双工通信：指数据可以在 一条线路同时沿两个方向传送，因此又被称为双向同时通信。
* 单工通信：指数据只能延单一方向传送，即发送方只能发送数据，接收方只能接收数据。
* 半双工同信：数据可以再一条线路沿两个方向传送，但是不能同时传送，现在存在的轮询(polling)和基于AJAX的长轮询(long-polling)两种数据通信方式就属于此。
2. [http://websocket.org/](http://websocket.org/) 一个专门介绍WebSocket的网站。
3. [http://www.oracle.com/webfolder/technetwork/tutorials/obe/java//WebSocket/WebSocket.html](http://www.oracle.com/webfolder/technetwork/tutorials/obe/java//WebSocket/WebSocket.html) 由oracle提供的一个在javaEE7中构建WebSocket应用的例子