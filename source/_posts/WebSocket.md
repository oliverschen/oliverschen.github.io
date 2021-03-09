---
title: WebSocket
date: 2021-03-08 23:20:34
tags: WebSocket
category: WebSocket
comments: true
---

![Photo by JustJon on wallhaven.cc](/websocket.png)


WebSocket 协议建立的是一个长连接，Http 协议是浏览器通过主动请求服务端建立连接获取数据，而 WebScoket 协议服务端可以实现主动向浏览器推送数据服务，浏览器也可以主动向服务端发生请求，它是一个全双工通信协议。WebSocket 协议在第一次连接时需要通过 Http 协议进行建立，建立成功之后通过 TCP 进行数据交换。在一些需要即使获取服务端信息的场景下，如果使用 Http 只能采用轮训的方式来获取，在及时性和资源方面都不友好，而通过 WebSocket 可以立即收到服务端的推送消息。

<!--more-->
### Socket

Socket：一组系统为了避免直接操作底层 TCP/UDP 协议而提供的 API 接口。在通过网络层 IP 协议找到对应的主机和应用程序。
WebSocket：和 Http 一样，是应用层协议。首次建立连接时需要通过 Http 进行连接，连接建立成功后通过 TCP 进行通讯。

### 使用
#### gradle 坐标

```
implementation 'org.springframework.boot:spring-boot-starter-websocket'
```

#### 启动类

开启 WebSocket 自动装配，和测试群发消息的 API

```
@RestController
@EnableWebSocket // 开启 WebSocket
@SpringBootApplication
public class WebSocketApplication {

    public static void main(String[] args) {
        SpringApplication.run(DemoGradleApplication.class, args);
    }

    @Autowired
    private WsService wsService;

    /**
     * 群发消息
     */
    @GetMapping("/publish/{msg}")
    public void publish(@PathVariable("msg") String msg) {
        wsService.publishMsg(msg);
    }
}
```

#### 配置类

```
@Configuration
public class WebScoketConfig {

    @Bean
    public ServerEndpointExporter serverEndpointExporter() {
        return new ServerEndpointExporter();
    }
}
```

#### WebSocket对外服务

这里需要注意使用 Spring 自动注入的时候只能在首次获取到注入值

```
@Slf4j
@Component
@ServerEndpoint(value = "/ws/conn")
public class WsServer implements InitializingBean {
    
    private static WsService wsService;

    /**
     * 建立webSocket连接
     */
    @OnOpen
    public void onOpen(Session session) {
        wsService.create(session);
    }

    /**
     * 接收到前端消息
     */
    @OnMessage
    public void onMessage(Session session, String message) {
        wsService.receiveMessage(session, message);
    }

    /**
     * 发生异常
     */
    @OnError
    public void onError(Session session, Throwable throwable) {
        log.error("发生异常，sessionId：{}", session.getUserPrincipal().getName());
    }

    /**
     * 关闭连接
     */
    @OnClose
    public void onClose(Session session) {
        wsService.close(session);
    }

    /**
     * 实际有用的是静态变量，自动注入的只是在 bean 初始化完成时赋值给静态变量
     */
    @Autowired
    private WsService wsServiceAuto;
    
    /**
     * WebSocket 是多例的，启动之后客户端访问时创建的新的实例中 wsService 为空，
     * 这里初始化完成之后设置一个全局静态变量
     */
    @Override
    public void afterPropertiesSet() throws Exception {
        wsService = this.wsServiceAuto;
    }
}
```

#### service类

```
@Slf4j
@Service
public class WsService {

    /**
     * 保存所有的WebSocket连接
     */
    private static final ConcurrentHashMap<Integer, Session> WEB_SOCKET_MAP = new ConcurrentHashMap<>();

    /**
     * 创建
     */
    public void create(Session session) {
        Integer sessionId = getSessionId(session);
        WEB_SOCKET_MAP.put(sessionId, session);
        log.info("新客户端{}连接成功", sessionId);
    }

    /**
     * 收到消息
     */
    public void receiveMessage(Session session, String message) {
        log.info("客户端{}收到消息{}", getSessionId(session), message);
        sendMessage(session, message.replace("吗？", "！"));
    }

    /**
     * 发送消息
     */
    private void sendMessage(Session session, String msg) {
        try {
            session.getBasicRemote().sendText(msg);
        } catch (IOException e) {
            log.error("error msg:", e);
        }
        log.info("发送消息成功：{}", msg);
    }

    /**
     * 关闭
     */
    public void close(Session session) {
        Integer sessionId = getSessionId(session);
        WEB_SOCKET_MAP.remove(sessionId);
        log.info("客户端{}关闭", sessionId);
    }

    /**
     * 群发消息
     */
    public void publishMsg(String msg) {
        WEB_SOCKET_MAP.forEach((key, value) -> sendMessage(value, msg));
    }

    public Integer getSessionId(Session session) {
        return Integer.valueOf(session.getId());
    }
}
```

### 测试

1. 使用[在线测试地址](http://www.websocket-test.com/)测试 WebSocket 连接是否能正常建立。
2. 多开窗口，访问 `http://localhost:7777/publish` 测试群发消息。


###

<center>别拖延，都要还回来</center>