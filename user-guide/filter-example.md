---
title: 消息过滤
---

###  消息过滤

很多情况下，**标签** 只是用来简单的选择用户需要的消息，例如：

```java

DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("CID_EXAMPLE");
consumer.subscribe("TOPIC", "TAGA || TAGB || TAGC");

```

这个消费者就会接收到`TAGA`、`TAGB`、`TAGC`中的信息，但是只能允许一个消息有一个标签，不能支持其他的复杂业务场景，
因此，我们可以使用`SQL`表达式来过滤消息

### 原则

`SQL` 的特点是可以通过一些消息中属性的计算。通过在`RocketMQ`中定义的语法，实现一些有意思的逻辑，例如：


```
------------
| message  |
|----------|  a > 5 AND b = 'abc'
| a = 10   |  --------------------> Gotten
| b = 'abc'|
| c = true |
------------


------------
| message  |
|----------|   a > 5 AND b = 'abc'
| a = 1    |  --------------------> Missed
| b = 'abc'|
| c = true |
------------
```


### 语法

`RocketMQ` 只定义了一些简单的语法来支持这些特性，当然使用者也可以对其进行一些简单的扩展

1. 数字类型的比较,如 `>`,`>=`,`<`,`<=`,`BETWEEN`,`=`
2. 字符类型的比较，如 `=`,`<>`,`IN`
3. `IS NULL` 或 `IS NOT NULL`
4. 逻辑运算 `AND`,`OR`,`NOT`


#### 常量类型

1. 数字 ，`123`,`3.1415`
2. 字符 ，`'abc'` 这里必须要使用单引号
3. `NULL` 特殊常量
4. 布尔类型 `TRUE` 或者 `FALSE`




### 使用限制

只有消费者在订阅消息的时候使用`SQL92`

```java
public void subscribe(final String topic, final MessageSelector messageSelector)
```

### 生产者范例

消息生产者可以在发送消息的时候通过`putUserProperty`放置一些属性
```java

DefaultMQProducer producer = new DefaultMQProducer("please_rename_unique_group_name");
producer.start();

Message msg = new Message("TopicTest",
    tag,
    ("Hello RocketMQ " + i).getBytes(RemotingHelper.DEFAULT_CHARSET)
);
// 设置属性
msg.putUserProperty("a", String.valueOf(i));

SendResult sendResult = producer.send(msg);

producer.shutdown();

```

### 消费者范例

消费者在接收消息的时候可以使用`MessageSelector.bySql`来进行消息的过滤

```java
DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("please_rename_unique_group_name_4");

// 只有订阅的消息包含属性 a, 并且 a >=0 and a <= 3
consumer.subscribe("TopicTest", MessageSelector.bySql("a between 0 and 3");

consumer.registerMessageListener(new MessageListenerConcurrently() {
    @Override
    public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> msgs, ConsumeConcurrentlyContext context) {
        return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
    }
});
consumer.start();

```
