# [分布式消息组件-Rabbitmq](../README.md)

- [RabbitMQ使用](#RabbitMQ使用)
  - [一 rabbitmq工作模式](#一-rabbitmq工作模式)
      - [1 rabbitmq架构图](#1-rabbitmq架构图)
      - [2 rabbitmq工作模式介绍](#2-rabbitmq工作模式介绍)
  - [二 rabbitmq确认机制](#二-rabbitmq确认机制)
  - [三 ttl+dlx实现延迟队列](#三-ttl&dlx实现延迟队列)

# 说明文档

| 文档版本|   版本   |  更新说明  |更新时间 | 更新人 |
| ---------|-------|-------|-------|------------ |
| v1.0.0|   **rabbitmq-3.8.5**  | 编写rabbitmq基础及高阶使用文档 | 2021/4/11 | 张子尧 |


# RabbitMQ使用

&nbsp;&nbsp;MQ全称 Message Queue（消息队列），是在消息的传输过程中保存消息的容器。多用于分布式系统之间进行通信。<br>
![img.png](../resources/images/mq.png)

##

**MQ优势**

- 应用解耦：提高系统容错性和可维护性
- 异步提速：提升用户体验和系统吞吐量
- 削峰填谷：提高系统稳定性
  <br>

**MQ劣势**

- 系统可用性降低：系统引入的外部依赖越多，系统稳定性越差。一旦 MQ 宕机，就会对业务造成影响。
- 系统复杂度提高： MQ的加入大大增加了系统的复杂度，以前系统间是同步的远程调用，现在是通过 MQ 进行异步调用。

##

**常见mq产品介绍图**
<br>  <br>
![img.png](../resources/images/mq产品介绍图.png)

## 一 rabbitmq工作模式

### 1 rabbitmq架构图

![img.png](../resources/images/rabbitmq架构图.png)

- virtual host：虚拟机，rabbitmq会把内部空间隔离出一个个的vhost，virtual host之间是完全隔离的，方便不同环境建立virtual host做数据隔离。
- connection：客户端和rabbitmq server之间的TCP连接。
-

channel：channel是在connection内部建立的逻辑连接通道，amqp包含了channelId帮助客户端识别channel，所以在每一个channel之间是完全隔离的。channel作为轻量级的connection，极大的减少了操作系统建立 TCP连接的开销

- exchange：生产者发送消息之后会首先到达exchange，然后根据规则匹配具体的routing key把消息分发达queue中，通常exchange包括fanout  direct  和topic三种模式。
    - fanout：广播
    - direct：定向
    - topic：通配符
- queue：exchange通过routing key最终会送达到queue中。
- binding：exchange和queue之间的虚拟连接，通常使用routing key 进行匹配绑定。消息通过不同的routing key路由到不同的队列当中；是消息分发的依据。

### 2 rabbitmq工作模式介绍

[rabbitmq官方说明提供了六种工作模式](https://www.rabbitmq.com/getstarted.html) : 简单模式  work queue  publish/subscribe  routing  topic  rpc。
<br>![img.png](../resources/images/rabbitmq工作模式.png)

+ 简单模式：生产者个消费者通过一个队列传输消息，消息会走默认的exchange
+ work queue：和简单模式一样，但消费者可以是多个。
+ publish/subscribe：发布订阅模式，生产者把消息发送到exchange，exchange在把消息路由到绑定该交换机的对列中，监听该队列的所有消费者都会接受到消息。
+ routing：消息在exchange上通过不同的routing key把消息分发到不同的队列。注意routing必须完全匹配才会路由到正确的队列中。
+ topic：该找匹配规格匹配routing key，*可以匹配一个单词，#可以匹配多个。
+ rpc：

## 二 rabbitmq确认机制

- confirm:生产者把消息发送到broker上时产生的状态，ack代表消息已经到达broker，nack则代表消息没有到达broker。
- return:代表broker正常confirm ack后，但消息没有进入具体的queue中则会产生的回调。

```java
public class Producer {
    /**
     * rabbitmq 确认机制 confirm&return
     * 开启confirm模式
     * connectionFactory.setPublisherConfirms(true);
     * 开启return模式
     * connectionFactory.setPublisherReturns(true);
     * @param rabbitTemplate {@link org.springframework.amqp.rabbit.core.RabbitTemplate}
     */
    public void confirm(RabbitTemplate rabbitTemplate) {
        rabbitTemplate.setConfirmCallback((confirm, ack, res) -> {
            log.info("confirm 方法被执行！！！");
            if (ack) {
                log.info("消息已经到达交换机：{}", res);
            } else {
                log.error("消息没有到达交换机：{}，返回信息为：{}", confirm, res);
            }
        });

        rabbitTemplate.setMandatory(true);
        rabbitTemplate.setReturnCallback((message, replyCode, replyText, exchange, routingKey) -> {
            log.info("return 执行了....");
            log.info("message:{}", message);
            log.info("replyCode:{}", replyCode);
            log.info("replyText:{}", replyText);
            log.info("exchange:{}", exchange);
            log.info("routingKey:{}", routingKey);
        });
        rabbitTemplate.convertAndSend(QueueConstants.AMQ_TOPIC, "email.qq.gaming", "hello queue!!!");

        try {
            Thread.sleep(5000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

## 三 ttl&dlx实现延迟队列

- ttl特性：设置消息过期时间，如果消息到了规定的时间内还没有被消费，则会被丢弃。
- dlx特性：消息在特定的时间内没有被消费则会被路由到新的exchange中，然后在路由的新的queue中。
- 延迟队列实现原理：通过ttl和dlx的特性可以设置两个对列，创建监听器监听死信队列，实现延迟队列。

**实现架构图**

![img.png](../resources/images/ttl+dlx.png)
<br>

**具体实现配置**

xml配置：

```xml
<!--==========================死信+ttl===========================================-->
<!--exchange-->
<rabbit:topic-exchange name="exchange_dlx" id="exchange_dlx" auto-declare="true">
    <rabbit:bindings>
        <rabbit:binding pattern="dlx.#" queue="queue_dlx"/>
    </rabbit:bindings>
</rabbit:topic-exchange>
        <!--dlx-->
<rabbit:queue id="queue_dlx" name="queue_dlx" auto-declare="true">
<rabbit:queue-arguments>
    <!--设置死信队列交换机-->
    <entry key="x-dead-letter-exchange" value="exchange_dlx_last"/>
    <!--设置死信队列路由key-->
    <entry key="x-dead-letter-routing-key" value="dlx.zhang"/>
    <!--设置队列过期时间 ttl-->
    <entry key="x-message-ttl" value="20000" value-type="java.lang.Integer"/>
    <!--设置队列长度限制-->
    <entry key="x-max-length" value="5" value-type="java.lang.Integer"/>
</rabbit:queue-arguments>
</rabbit:queue>
        <!---->
<rabbit:queue id="queue_dlx_last" name="queue_dlx_last"/>
<rabbit:topic-exchange name="exchange_dlx_last" auto-declare="true">
<rabbit:bindings>
    <rabbit:binding pattern="dlx.#" queue="queue_dlx_last"/>
</rabbit:bindings>
</rabbit:topic-exchange>
        <!--==========================死信+ttl===========================================-->
```

配置文件配置：

```java
/**
 * @author zhangziyao
 * @date 2020/12/26 9:10 下午
 */
@Component
@Slf4j
public class RabbitAdminConfig implements InitializingBean {
    @Autowired
    private RabbitAdmin rabbitAdmin;

    /**
     * 创建队列
     *
     * @param queueName 队列名称
     * @return 返回队列名称
     */
    public String createQueue(String queueName, Map<String, Object> ages) {
        log.info("创建队列：{}", queueName);
        rabbitAdmin.declareQueue(new Queue(queueName, true, false, false, ages));
        return queueName;
    }

    /**
     * 创建交换机
     *
     * @param ExchangeName 交换机名称
     * @param workMode     交换机工作模式 参考 {@link WorkMode}
     * @return 返回交换机名称
     */
    public String createExchange(String ExchangeName, WorkMode workMode) {
        log.info("创建交换机：{}", ExchangeName);
        switch (workMode) {
            case fanout:
                rabbitAdmin.declareExchange(new FanoutExchange(ExchangeName, true, false, null));
                break;
            case direct:
                rabbitAdmin.declareExchange(new DirectExchange(ExchangeName, true, false, null));
                break;
            case topic:
                rabbitAdmin.declareExchange(new TopicExchange(ExchangeName, true, false, null));
                break;
            default:
                log.info("没有要创建的交换机类型：{}", workMode);
        }
        return ExchangeName;
    }

    @Override
    public void afterPropertiesSet() throws Exception {

        Map<String, Object> agrs = new HashMap<>();
        agrs.put(ArgKeys.X_MESSAGE_TTL.getKey(), 10000);
        agrs.put(ArgKeys.X_DEAD_LETTER_EXCHANGE.getKey(), "dlx.exchange.last.admin");
        agrs.put(ArgKeys.X_DEAD_LETTER_ROUTING_KEY.getKey(), "dlx.zhang.admin");
        agrs.put(ArgKeys.X_MAX_LENGTH.getKey(), 5);


        rabbitAdmin.declareBinding(new Binding(createQueue("dlx.queue.admin", agrs)
                , Binding.DestinationType.QUEUE
                , createExchange("dlx.exchange.admin", WorkMode.topic)
                , "dlx.#"
                , null));
        rabbitAdmin.declareBinding(new Binding(createQueue("dlx.queue.last.admin", null)
                , Binding.DestinationType.QUEUE
                , createExchange("dlx.exchange.last.admin", WorkMode.topic)
                , "dlx.#"
                , null));
    }
}
```