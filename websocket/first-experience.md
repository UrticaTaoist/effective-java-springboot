# First Experience

### 1.POM配置

```markup
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-websocket</artifactId>
        <exclusions>
            <exclusion>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-tomcat</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-undertow</artifactId>
    </dependency>
</dependencies>
```

### 2.下面是直接搭建服务器。。

```java
package cn.zhenyang.socket.server.controller.socket;

import cn.zhenyang.socket.server.config.GetHttpSessionConfigurator;
import cn.zhenyang.socket.server.config.ServerDecoder;
import cn.zhenyang.socket.server.config.ServerEncoder;
import cn.zhenyang.socket.server.constant.Params;
import org.springframework.stereotype.Component;
import org.springframework.web.context.request.RequestContextHolder;
import org.springframework.web.context.request.ServletRequestAttributes;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpSession;
import javax.websocket.*;
import javax.websocket.server.ServerEndpoint;
import java.io.IOException;
import java.util.concurrent.CopyOnWriteArraySet;

/**
 * @Author ZhenYang
 * @Date Created in 2018/2/19 3:58
 * @Description
 */

@ServerEndpoint(value = "/websocket",configurator = GetHttpSessionConfigurator.class/*,decoders = ServerDecoder.class,encoders = ServerEncoder.class*/)
@Component
public class WebSocketServer {
    //静态变量，用来记录当前在线连接数。应该把它设计成线程安全的。
    public static int onlineCount = 0;
    //concurrent包的线程安全Set，用来存放每个客户端对应的MyWebSocket对象。
    public static CopyOnWriteArraySet<WebSocketServer> webSocketSet = new CopyOnWriteArraySet<WebSocketServer>();
    //与某个客户端的连接会话，需要通过它来给客户端发送数据
    public Session session;




    /**
     * 连接建立成功调用的方法
     */
    @OnOpen
    public void onOpen(Session session,EndpointConfig config) {
        this.session = session;

        System.out.println("the session is : "+session.getId());

        System.out.println("session content is : "+session.getId());
        webSocketSet.add(this);     //加入set中
        addOnlineCount();           //在线数加1
        System.out.println("有新连接加入！当前在线人数为" + getOnlineCount());
//        ServletActionContext servletActionContext ;
        try {
            //sendMessage(CommonConstant.CURRENT_WANGING_NUMBER.toString());
//            System.out.println(Params.httpSession.getAttribute("xxx"));
            sendMessage("22222222");
        } catch (IOException e) {
            System.out.println("IO异常");
        }
    }

    /**
     * 连接关闭调用的方法
     */
    @OnClose
    public void onClose() {
        System.out.println(this.session.getId()+":这个断开");
        webSocketSet.remove(this);  //从set中删除
        subOnlineCount();           //在线数减1
        System.out.println("有一连接关闭！当前在线人数为" + getOnlineCount());
    }

    /**
     * 收到客户端消息后调用的方法
     *
     * @param message 客户端发送过来的消息
     */
    @OnMessage
    public void onMessage(String message, Session session) {
//        HttpServletRequest request =
//        System.out.println(httpSession.getAttribute("xxx"));
        System.out.println("来自客户端的消息:" + message);
        System.out.println("onMessage sessionId is : "+session.getId());
        //群发消息
        for (WebSocketServer item : webSocketSet) {
            try {
                item.sendMessage(message);
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }

    /**
     * 发生错误时调用
     */
    @OnError
    public void onError(Session session, Throwable error) {
        System.out.println(error.getMessage());
    }

    /**
     * 给当前用户发送消息
     * @param message
     * @throws IOException
     */
    public void sendMessage(String message) throws IOException {
        System.out.println(Params.httpSession.getAttribute("xxx"));
        this.session.getBasicRemote().sendText(message+" : sendMessage id is : "+this.session.getId());
        //this.session.getAsyncRemote().sendText(message);
    }

    /**
     * 群发自定义消息
     */
    public static void sendInfo(String message) throws IOException {
        for (WebSocketServer item : webSocketSet) {
            try {
                item.sendMessage(message);
            } catch (IOException e) {
                continue;
            }
        }
    }

    public static synchronized int getOnlineCount() {
        return onlineCount;
    }

    public static synchronized void addOnlineCount() {
        WebSocketServer.onlineCount++;
    }

    public static synchronized void subOnlineCount() {
        WebSocketServer.onlineCount--;
    }
}
```

另声明了一个静态会话。

```text
public static HttpSession httpSession;
```

相关配置：

```java
@Component
public class GetHttpSessionConfigurator extends ServerEndpointConfig.Configurator {
    @Override
    public void modifyHandshake(ServerEndpointConfig sec, HandshakeRequest request, HandshakeResponse response) {
        HttpSession httpSession = (HttpSession) request.getHttpSession();
        //解决httpSession为null的情况
        if (httpSession == null) {
            HttpServletRequest request2 = ((ServletRequestAttributes) RequestContextHolder.getRequestAttributes()).getRequest();
            httpSession = request2.getSession();
//            httpSession = Params.httpSession;
        }
        sec.getUserProperties().put(HttpSession.class.getName(), httpSession);
    }
}
```



```text
/**
 *
 * @Author ZhenYang
 * @Date Created in 2018/1/31 9:13
 * @Description
 * 使用@ServerEndpoint创立websocket endpoint
 * 首先要注入 ServerEndpointExporter，
 * 这个bean会自动注册使用了 @ServerEndpoint 注
 * 解声明的 Web Socket endpoint。
 * 要注意，如果使用独立的 Servlet 容器，
 * 而不是直接使用 Spring Boot 的内置容器，
 * 就不要注入 ServerEndpointExporter，
 * 因为它将由容器自己提供和管理
 */
@Configuration
//@EnableWebSocket
//public class WebSocketConfig implements WebSocketConfigurer {
public class WebSocketConfig {

    @Bean
    public ServerEndpointExporter serverEndpointExporter(){
        return new ServerEndpointExporter();
    }
//@Override
//public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
//    registry.addHandler(myHandler(),"/websocket")
//            .addInterceptors(new HttpSessionHandshakeInterceptor())
//    ;
//}
//    @Bean
//    public WebSocketHandler myHandler(){
//        return new MyHandler();
//    }
}
```

### 3.搭建一下客户端

客户端就这一个类

```text
package cn.zhenyang.socket.client.controller.rest;

import cn.zhenyang.common.websocket.constant.BasicParameter;
import cn.zhenyang.common.websocket.utils.WebSocketClientUtil;
import cn.zhenyang.socket.client.entity.User;
import com.alibaba.fastjson.JSON;
import org.java_websocket.client.WebSocketClient;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.util.NestedServletException;

/**
 * @Author ZhenYang
 * @Date Created in 2018/2/19 23:48
 * @Description
 */
@RestController
@RequestMapping("/")
public class WebsocketClientRest {
    public static WebSocketClient client;

    @RequestMapping("init")
    public void initConnect() {
        client = WebSocketClientUtil.getWebSocketClient(BasicParameter.WebSocketServerURI);
        System.out.println(client.getDraft());
//        while (!client.getReadyState().equals(WebSocket.READYSTATE.OPEN)) {
//            System.out.println("还没有打开");
//        }
//        System.out.println("打开了");
    }

    @PostMapping("send")
    public void send() {
//        String str = JSON.toJSONString(user);
        try {
            client.send("asdfsadfdsfas");
        }catch (Exception e){
            System.out.println("连接失败");
        }

    }
    @RequestMapping("closeSocket")
    public void close(){
        client.close();
    }
}
```

也涉及一个静态变量，用于指定服务器地址

```text
public static final String WebSocketServerURI = "ws://localhost:8081/websocket";
```



