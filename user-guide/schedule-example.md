---

title: 延时消息
---

### 什么是延时消息

延时消息与普通消息的区别就在于发送消息的时候需要提供一个延迟时间


### 程序

1. 启动消息消费者等待被订阅的消息到来

```java
import org.apache.rocketmq.client.consumer.DefaultMQPushConsumer;
import org.apache.rocketmq.client.consumer.listener.ConsumeConcurrentlyContext;
import org.apache.rocketmq.client.consumer.listener.ConsumeConcurrentlyStatus;
import org.apache.rocketmq.client.consumer.listener.MessageListenerConcurrently;
import org.apache.rocketmq.common.message.MessageExt;
import java.util.List;

public class ScheduledMessageConsumer {

 public static void main(String[] args) throws Exception {
     // 初始化消费者
     DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("ExampleConsumer");
     // 订阅主题
     consumer.subscribe("TestTopic", "*");
     // 注册监听
     consumer.registerMessageListener(new MessageListenerConcurrently() {
         @Override
         public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> messages, ConsumeConcurrentlyContext context) {
             for (MessageExt message : messages) {
                 // 打印信息
                 System.out.println("Receive message[msgId=" + message.getMsgId() + "] "
                         + (System.currentTimeMillis() - message.getStoreTimestamp()) + "ms later");
             }
             return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
         }
     });
     // 启动消费者实例
     consumer.start();
 }
}
```
2. 发送延时消息

```java

import org.apache.rocketmq.client.producer.DefaultMQProducer;
import org.apache.rocketmq.common.message.Message;

public class ScheduledMessageProducer {

 public static void main(String[] args) throws Exception {
     // 初始化延时消息生产者
     DefaultMQProducer producer = new DefaultMQProducer("ExampleProducerGroup");
     // 启动消息生产者
     producer.start();
     int totalMessagesToSend = 100;
     for (int i = 0; i < totalMessagesToSend; i++) {
         Message message = new Message("TestTopic", ("Hello scheduled message " + i).getBytes());
         // 这条消息将会被延迟10秒发送
         message.setDelayTimeLevel(3);
         // 发送消息
         producer.send(message);
     }

     // 关闭消息生产者实例
     producer.shutdown();
 }

}
```
