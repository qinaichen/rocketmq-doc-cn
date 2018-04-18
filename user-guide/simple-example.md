---
title: 简单范例
---

### 简单消息范例

使用`RocketMQ`发送消息有三种方式

- 同步可靠
- 异步可靠
- 单向传输

接下来通过代码示例来介绍这三种方式，我们可以根据这三种范例来选择适合自己业务需要的方式

#### 同步可靠传输

同步可靠传输被用于大量的业务场景中，如重要的消息提醒、短信提醒、短信营销系统等
```java
public class SyncProducer {
    public static void main(String[] args) throws Exception {
        //使用producer组名初始化一个producer实例
        DefaultMQProducer producer = new
            DefaultMQProducer("please_rename_unique_group_name");
        //启动实例
        producer.start();
        for (int i = 0; i < 100; i++) {
            //创建一个消息实例, 指定主题、标签、消息体.
            Message msg = new Message("TopicTest" /* 消息主题 */,
                "TagA" /* 消息标签 */,
                ("Hello RocketMQ " +
                    i).getBytes(RemotingHelper.DEFAULT_CHARSET) /* 消息体 */
            );
            //把消息发送到brokers中的一个broker
            SendResult sendResult = producer.send(msg);
            System.out.printf("%s%n", sendResult);
        }
        //如果这个producer不再使用，将其关闭
        producer.shutdown();
    }
}
```

#### 异步可靠传输

异步可靠传输大量的用于对响应时间敏感的业务场景中

```java
public class AsyncProducer {
    public static void main(String[] args) throws Exception {
        //使用producer组名初始化一个producer实例
        DefaultMQProducer producer = new DefaultMQProducer("ExampleProducerGroup");
        //启动实例
        producer.start();
        producer.setRetryTimesWhenSendAsyncFailed(0);
        for (int i = 0; i < 100; i++) {
                final int index = i;
                //创建一个消息实例, 指定主题、标签、消息体.
                Message msg = new Message("TopicTest",
                    "TagA",
                    "OrderID188",
                    "Hello world".getBytes(RemotingHelper.DEFAULT_CHARSET));
                producer.send(msg, new SendCallback() {
                    @Override
                    public void onSuccess(SendResult sendResult) {
                        System.out.printf("%-10d OK %s %n", index,
                            sendResult.getMsgId());
                    }
                    @Override
                    public void onException(Throwable e) {
                        System.out.printf("%-10d Exception %s %n", index, e);
                        e.printStackTrace();
                    }
                });
        }
        //如果这个producer不再使用，将其关闭
        producer.shutdown();
    }
}
```

#### 单向传输

单向传输被用在可靠稳定的业务中，例如 日志收集

```java
public class OnewayProducer {
    public static void main(String[] args) throws Exception{
        //使用producer组名初始化一个producer实例
        DefaultMQProducer producer = new DefaultMQProducer("ExampleProducerGroup");
        //启动实例
        producer.start();
        for (int i = 0; i < 100; i++) {
            //创建一个消息实例, 指定主题、标签、消息体.
            Message msg = new Message("TopicTest" /* 主题 */,
                "TagA" /* 标签 */,
                ("Hello RocketMQ " +
                    i).getBytes(RemotingHelper.DEFAULT_CHARSET) /* 消息体*/
            );
            //把消息发送到brokers中的一个broker
            producer.sendOneway(msg);

        }
        //如果这个producer不再使用，将其关闭
        producer.shutdown();
    }
}
```
