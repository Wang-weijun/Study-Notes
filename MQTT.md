## 使用 WebSocket 连接 MQTT

WebSocket 使得客户端和服务器之间的数据交换变得更加简单，允许服务端主动向客户端推送数据。在 WebSocket API 中，浏览器和服务器只需要完成一次握手，两者之间就直接可以创建持久性的连接，并进行双向数据传输。



## 两款客户端比较

### Paho.mqtt.js

[Paho](https://www.eclipse.org/paho/) 是 Eclipse 的一个 MQTT 客户端项目，Paho JavaScript Client 是其中一个基于浏览器的库，它使用 WebSockets 连接到 MQTT 服务器。相较于另一个 JavaScript 连接库来说，其功能较少，不推荐使用。



### MQTT.js

[MQTT.js](https://github.com/mqttjs/MQTT.js) 是一个完全开源的 MQTT 协议的客户端库，使用 JavaScript 编写，可用于 Node.js 和浏览器。在 Node.js 端可以通过全局安装使用命令行连接，同时支持 MQTT/TCP、MQTT/TLS、MQTT/WebSocket 连接；值得一提的是 MQTT.js 还对[微信小程序](https://www.emqx.com/zh/blog/how-to-use-mqtt-in-wechat-miniprogram)有较好的支持。

本文将使用 MQTT.js 库进行 WebSocket 的连接讲解。









## 安装 MQTT.js

机器上装有 Node.js 运行环境，可直接使用 npm 命令安装 MQTT.js。

~~~javascript
npm install mqtt --save
~~~





## 连接至 MQTT 服务器

本文将使用 EMQX 提供的 [免费公共 MQTT 服务器](https://www.emqx.com/zh/mqtt/public-mqtt5-broker)，该服务基于 EMQX 的 [MQTT 物联网云平台](https://www.emqx.com/zh/cloud) 创建。服务器接入信息如下：

- Broker: **broker.emqx.io**
- TCP Port: **1883**
- Websocket Port: **8083**



> EMQX 使用 8083 端口用于普通连接，8084 用于 SSL 上的 WebSocket 连接。



为了简单起见，让我们将订阅者和发布者放在同一个文件中：



```javascript
const clientId = 'mqttjs_' + Math.random().toString(16).substr(2, 8)

const host = 'ws://broker.emqx.io:8083/mqtt'

const options = {
  keepalive: 60,
  clientId: clientId,
  protocolId: 'MQTT',
  protocolVersion: 4,
  clean: true,
  reconnectPeriod: 1000,
  connectTimeout: 30 * 1000,
  will: {
    topic: 'WillMsg',
    payload: 'Connection Closed abnormally..!',
    qos: 0,
    retain: false
  },
}

console.log('Connecting mqtt client')
const client = mqtt.connect(host, options)

client.on('error', (err) => {
  console.log('Connection error: ', err)
  client.end()
})

client.on('reconnect', () => {
  console.log('Reconnecting...')
})
```









## 连接地址

> 上文示范的连接地址可以拆分为： `ws:` // `broker` . `emqx.io` : `8083` `/mqtt`
>
> 即 `协议` // `主机名` . `域名` : `端口` / `路径`

-------

**常见错误**

* 连接地址没有指明协议：WebSocket 作为一种通信协议，其使用 `ws` (非加密)、`wss`(SSL 加密) 作为协议标识。MQTT.js 客户端支持多种协议，连接地址需指明协议类型；

* 连接地址没有指明端口：MQTT 并未对 WebSocket 接入端口做出规定，EMQX 上默认使用 `8083`/ `8084` 分别作为**非加密连接**、**加密连接**端口。而 WebSocket 协议默认端口同 HTTP 保持一致 (80/443)，不填写端口则表明使用 WebSocket 的默认端口连接；而使用标准 MQTT 连接时则无需指定端口，如 MQTT.js 在 Node.js 端可以使用 `mqtt://localhost` 连接至标准 MQTT 1883 端口，当连接地址是 `mqtts://localhost` 则连接到 8884 端口；

* 连接地址无路径：MQTT-WebSoket 统一使用 `/path` 作为连接路径，连接时需指明，在 EMQX 上使用的路径为 `/mqtt`；
* 协议与端口不符：使用了 `wss` 连接却连接到 `8083` 端口；
* 在 HTTPS 下使用非加密的 WebSocket 连接： Google 等机构在推进 HTTPS 的同时也通过浏览器约束进行了安全限定，即 HTTPS 连接下浏览器会自动禁止使用非加密的 `ws` 协议发起连接请求；
* 证书与连接地址不符： 启用 **SSL/TLS**  加密连接。

*****







## 连接选项

上面代码中， `options` 是客户端连接选项，以下是主要参数说明。

----

* keepalive：心跳时间，默认 60秒，设置 0 为禁用；

- clientId： 客户端 ID ，不可重复；
- username：连接用户名（可选）；
- password：连接密码（可选）；
- clean：true，设置为 false 以在离线时接收 QoS 1 和 2 消息；
- reconnectPeriod：默认 1000 毫秒，两次重新连接之间的间隔，客户端 ID 重复、认证失败等客户端会重新连接；
- connectTimeout：默认 30 * 1000毫秒，收到 CONNACK 之前等待的时间，即连接超时时间；
- will：遗嘱消息，当客户端严重断开连接时，Broker 将自动发送的消息。 一般格式为：
  - topic：要发布的主题
  - payload：要发布的消息
  - qos：QoS
  - retain：保留标志

---







## 订阅/取消订阅

连接成功之后才能订阅，且订阅的主题必须符合 MQTT 订阅主题规则；

注意 JavaScript 的异步非阻塞特性，只有在 connect 事件后才能确保客户端已成功连接，或通过 `client.connected` 判断是否连接成功：

~~~javascript
client.on('connect', () => {
  console.log('Client connected:' + clientId)
  // Subscribe
  client.subscribe('testtopic', { qos: 0 })
})
~~~

~~~javascript
// Unsubscribe
client.unubscribe('testtopic', () => {
  console.log('Unsubscribed')
})
~~~



## 发布/接收消息

发布消息到某主题，发布的主题必须符合 MQTT 发布主题规则，否则将断开连接。发布之前无需订阅该主题，但要确保客户端已成功连接：

```javascript
// Publish
client.publish('testtopic', 'ws connection demo...!', { qos: 0, retain: false })
```

~~~javascript
// Received
client.on('message', (topic, message, packet) => {
  console.log('Received Message: ' + message.toString() + '\nOn topic: ' + topic)
})
~~~





# SpringBoot集成Mqtt

## 导入依赖

~~~xml
<!--mqtt中间件-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-integration</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.integration</groupId>
    <artifactId>spring-integration-stream</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.integration</groupId>
    <artifactId>spring-integration-mqtt</artifactId>
</dependency>
~~~

## 编写配置类

~~~yaml
mqtt:
  hostUrl: tcp://127.0.0.1:1883
  username: guest
  password: 123456
  keepAliveInterval: 60
  connectionTimeout: 10
  client-id: mqttClient01
  server-id: mqttServer01
  data-topic: /wvp_ptz
  will-topic: /will
  will-content: mqttServer Unexpected disconnection
  completion-timeout: 10000
~~~

## 编写配置类

~~~java
@Configuration
@IntegrationComponentScan
@Data
public class MqttConfig {
 
    /**
     * mqtt地址
     */
    @Value("${mqtt.hostUrl}")
    private String hostUrl;
 
    /**
     * 用户名
     */
    @Value("${mqtt.username}")
    private String username;
 
    /**
     * 密码
     */
    @Value("${mqtt.password}")
    private String password;
 
    /**
     * 心跳间隔
     */
    @Value("${mqtt.keepAliveInterval}")
    private int keepAliveInterval;
 
    /**
     *连接超时
     */
    @Value("${mqtt.connectionTimeout}")
    private int connectionTimeout;
 
    /**
     *客户端id
     */
    @Value("${mqtt.client-id}")
    private String clientId;
 
    /**
     *服务端id
     */
    @Value("${mqtt.server-id}")
    private String serverId;
 
    /**
     *消息主题
     */
    @Value("${mqtt.data-topic}")
    private String dataTopic;
 
    /**
     *遗愿主题
     */
    @Value("${mqtt.will-topic}")
    private String willTopic;
 
    /**
     *遗愿消息内容
     */
    @Value("${mqtt.will-content}")
    private String willContent;
 
    /**
     *断开连接超时时间
     */
    @Value("${mqtt.completion-timeout}")
    private long completionTimeout;
 
 
    @Bean
    public MqttConnectOptions getMqttConnectOptions(){
        MqttConnectOptions options = new MqttConnectOptions();
        // 设置发布端地址,多个用逗号分隔, 如:tcp://111:1883,tcp://222:1883
        // 当第一个111连接上后,222不会在连,如果111挂掉后,重试连111几次失败后,会自动去连接222
        options.setServerURIs(hostUrl.split(","));
        options.setKeepAliveInterval(keepAliveInterval);
        options.setUserName(username);
        options.setPassword(password.toCharArray());
        options.setConnectionTimeout(connectionTimeout);
        // 设置“遗嘱”消息的话题，若客户端与服务器之间的连接意外中断，服务器将发布客户端的“遗嘱”消息。
        options.setWill(willTopic,(clientId + willContent).getBytes(),1,true);
        // 设置是否清空session,这里如果设置为false表示服务器会保留客户端的连接记录，
        // 把配置里的 cleanSession 设为false，客户端掉线后 服务器端不会清除session，
        // 当重连后可以接收之前订阅主题的消息。当客户端上线后会接受到它离线的这段时间的消息
        options.setCleanSession(true);
        options.setMaxInflight(1000);
 
        //设置断开重新连接
        options.setAutomaticReconnect(true);
        return options;
    }
 
    @Bean
    public MqttPahoClientFactory mqttPahoClientFactory(){
        DefaultMqttPahoClientFactory clientFactory = new DefaultMqttPahoClientFactory();
        clientFactory.setConnectionOptions(getMqttConnectOptions());
        return clientFactory;
    }
 
 
    /**
     * 发送通道
     */
    @Bean
    public MessageChannel mqttOutBoundChannel(){
        return new DirectChannel();
    }
 
    /**
     * 发送消息配置
     * @return
     */
    @Bean
    @ServiceActivator(inputChannel = "mqttOutBoundChannel")
    public MessageHandler messageHandler(){
        //clientId每个连接必须唯一,否则,两个相同的clientId相互挤掉线
        //String clientIdStr = clientId + new SecureRandom().nextInt(10);
        MqttPahoMessageHandler mqttPahoMessageHandler = new MqttPahoMessageHandler(clientId,mqttPahoClientFactory());
        //async如果为true，则调用方不会阻塞。而是在发送消息时等待传递确认。默认值为false（发送将阻塞，直到确认发送)
        mqttPahoMessageHandler.setAsync(true);
        mqttPahoMessageHandler.setDefaultTopic(dataTopic);
        mqttPahoMessageHandler.setDefaultQos(1);
        mqttPahoMessageHandler.setAsyncEvents(true);
        return mqttPahoMessageHandler;
    }
 
    /**
     * 接收通道
     */
    @Bean
    public MessageChannel mqttInboundChannel(){
        return new DirectChannel();
    }
 
    /**
     * 配置监听的 topic 支持通配符
     * @return
     */
    @Bean
    public MessageProducer inBound(){
        String[] topics = new String[]{
                "$SYS/brokers/+/clients/+/disconnected", //客户端下线主题
                "$SYS/brokers/+/clients/+/connected",    //客户端上线主题
               // "$exclusive/",                           //排它策略前缀
                dataTopic,
                willTopic
        };
        //serverId每个连接必须唯一,否则,两个相同的serverId相互挤掉线
        //String serverIdStr = serverId + new SecureRandom().nextInt(10);
        MqttPahoMessageDrivenChannelAdapter adapter =
                new MqttPahoMessageDrivenChannelAdapter(serverId,mqttPahoClientFactory(),topics);
        adapter.setOutputChannel(mqttInboundChannel());
        adapter.setDisconnectCompletionTimeout(completionTimeout);
        adapter.setConverter(new DefaultPahoMessageConverter());
        adapter.setQos(1);
        return adapter;
    }
 
    /**
     * 通过通道获取数据
     * @return
     */
    @Bean
    @ServiceActivator(inputChannel = "mqttInboundChannel")
    public MessageHandler getInbound(){
        MessageHandler messageHandler = new MessageHandler() {
            @Override
            public void handleMessage(Message<?> message) throws MessagingException {
                String topic = message.getHeaders().get(MqttHeaders.RECEIVED_TOPIC).toString();
                String payload = message.getPayload().toString();
                //消息处理
                //MyMessageHandler.msgHandler(topic,payload);
                System.out.println("消息主题："+topic+",消息内容："+payload);
                //处理订阅topic:(data/#)到的所有的数据
            }
        };
        return messageHandler;
    }
 
 
    /**
     * 连接失败会触发
     * @param event
     */
    @EventListener(classes = MqttConnectionFailedEvent.class)
    public void mqttConnectFailedEvent(MqttConnectionFailedEvent event){
 
        System.out.println("mqttConnectionFailedEvent连接mqtt失败: "+event.toString());
    }
 
    /**
     * @desc 当async和async事件(async-events)都为true时,将发出MqttMessageSentEvent
     * 它包含消息、主题、客户端库生成的消息id、clientId和clientInstance（每次连接客户端时递增）
     * @date 2021/7/22
     *@param event
     * @return void
     */
    @EventListener(classes = MqttMessageSentEvent.class)
    public void mqttMessageSentEvent(MqttMessageSentEvent event){
        System.out.println("发送消息"+event.toString());
    }
 
    /**
     * @desc 当async和async事件(async-events)都为true时,将发出MqttMessageDeliveredEvent
     * 当客户端库确认传递时，将发出MqttMessageDeliveredEvent。它包含messageId、clientId和clientInstance，使传递与发送相关。
     * @date 2021/7/22
     *@param event
     * @return void
     */
    @EventListener(classes = MqttMessageDeliveredEvent.class)
    public void mqttMessageDeliveredEvent(MqttMessageDeliveredEvent event){
        System.out.println("消息发送成功："+event.toString());
    }
 
    /**
     * @desc 成功订阅到主题，MqttSubscribedEvent事件就会被触发(多个主题,多次触发)
     * @date 2021/7/22
     *@param event
     * @return void
     */
    @EventListener(classes = MqttSubscribedEvent.class)
    public void mqttSubscribedEvent(MqttSubscribedEvent event){
        System.out.println("消息订阅成功："+event.toString());
    }
}
~~~

















