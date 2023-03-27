# 卡夫卡交易

> 原文：<https://itnext.io/kafka-transaction-56f022af1b0c?source=collection_archive---------2----------------------->

![](img/ad5f834f066f53a0b80cd2f47f4cfd20.png)

安妮·斯普拉特在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上的照片

根据我的经验，在大多数情况下，使用 Kafka 处理至少一次或最多一次消息事件就足够了。

在卡夫卡那里实现事务性处理并不容易，因为它不是为事务性而生的，我想。

接下来，让我们看看为什么很难在使用 Kafka Streams 的应用程序中获得完整的事务处理，Kafka Streams 可以在您的流应用程序中进行复杂的处理。

kafka streams 应用程序可以包含许多消费-流程-生产的处理周期。使用 Kafka Streams 应用程序，您可以多次遇到以下情况:

*   如果您在应用程序中使用 Kafka Streams，可能会有很多类似于`map(), through(), transform(), flatMap(), etc`的函数调用，其中会发生重新划分，也就是说，带有中间主题的新主题将使用新主题关键字创建。
*   如果您的 kafka streams 应用程序是有状态的，本地状态将被异步同步到 changelog 主题。
*   如果在 kafka streams 应用程序的处理过程中，您必须将处理结果保存到外部数据库，那么您也必须考虑将偏移量保存到同一个外部数据库。

正如在上面的情况中所看到的，可能有许多消费、处理、生产过程，有许多不同的数据存储，如 kafka、本地存储和外部数据库，其中所有的事务都必须以原子的方式提交。但是在 kafka streams 应用程序中，很难以原子方式实现整个事务提交。

相反，它应该被划分为个人消费-加工-生产循环。

现在，我将向您展示如何在 Kafka 中进行生产者端事务和消费者端事务，以实现一次性处理:

*   在生产者端事务中，kafka 生产者使用 kafka 事务 api 发送带有事务配置的 avro 消息。
*   在消费者端事务中，kafka 消费者消费来自主题的 avro 消息，处理它们，将处理后的结果保存到外部数据库，偏移也保存到相同的外部数据库，最后所有数据库事务将以原子方式提交。

本文的完整代码可以在我的 github repo 中找到:

[](https://github.com/mykidong/kafka-transaction-example) [## my kidong/Kafka-交易-示例

### 此时您不能执行该操作。您已使用另一个标签页或窗口登录。您已在另一个选项卡中注销，或者…

github.com](https://github.com/mykidong/kafka-transaction-example) 

# 交易型卡夫卡制片人

我们先来讨论卡夫卡中的生产者端交易。

可以设置生成器的以下事务属性:

```
 // transaction properties.
        props.put(ProducerConfig.TRANSACTIONAL_ID_CONFIG, transactionalId); // unique transactional id.
        props.put(ProducerConfig.ENABLE_IDEMPOTENCE_CONFIG, true);
        props.put(ProducerConfig.ACKS_CONFIG, "all");
        props.put(ProducerConfig.RETRIES_CONFIG, 3);
       props.put(ProducerConfig.MAX_IN_FLIGHT_REQUESTS_PER_CONNECTION, 1);
        props.put(ProducerConfig.MAX_BLOCK_MS_CONFIG, 600000);
```

*   `ProducerConfig.TRANSACTIONAL_ID_CONFIG`必须是单个 kafka 生产者的唯一交易 id。
*   `ProducerConfig.ENABLE_IDEMPOTENCE_CONFIG`一定是`true`。
*   `ProducerConfig.ACKS_CONFIG`是`all`，即所有的领导者和追随者经纪人都向日志提交了消息。
*   `ProducerConfig.RETRIES_CONFIG`应大于 1 的尝试请求多次失败请求。
*   `ProducerConfig.MAX_IN_FLIGHT_REQUESTS_PER_CONNECTION`为 1，表示将保证有序的顺序。

让我们看看 kafka producer 以事务方式发送的消息:

```
*// construct producer.* KafkaProducer<UserKey, Events> producer = **new** KafkaProducer<>(props);

*// initiate transaction.* producer.initTransactions();
*log*.info(**"tx init..."**);
**try** {
    *// begin transaction.* producer.beginTransaction();
    *log*.info(**"tx begun..."**);

    **for**(**int** i = 0; i < 20; i++) {
        Events events = **new** Events();
        events.setCustomerId(**"customer-id-"** + (i % 5));
        events.setOrderInfo(**"some order info "** + **new** Date().toString() + **"-"** + i);

        Date date = **new** Date();
        events.setEventTime(date.getTime());

        UserKey key = **new** UserKey(events.getCustomerId().toString(), date);

        *// send messages.* Future<RecordMetadata> response = producer.send(**new** ProducerRecord<UserKey, Events>(topic, key, events));
        *log*.info(**"message sent ... "** + **new** Date().toString() + **"-"** + i);

        RecordMetadata recordMetadata = response.get();
        *log*.info(**"response - topic [{}], partition [{}], offset [{}]"**, Arrays.*asList*(recordMetadata.topic(), recordMetadata.partition(), recordMetadata.offset()).toArray());
    }

    *// commit transaction.* producer.commitTransaction();
    *log*.info(**"tx committed..."**);

} **catch** (KafkaException e) {
    *// For all other exceptions, just abort the transaction and try again.* producer.abortTransaction();
}

*// close producer.* producer.close();
```

*   在发送消息之前，必须初始化 Kafka producer 事务:`producer.initTransactions()`
*   发送消息后，提交事务:`producer.commitTransaction()`
*   如果出现异常，中止交易:`producer.abortTransaction()`

# 交易型卡夫卡消费者

现在，让我们看看消费者端的交易。

在我的场景中，您必须消费、处理消息，并将处理后的结果保存到外部数据库。同时，偏移量也必须保存到相同的外部数据库中。

在开始使用消息之前，让我们看看 db 表`offset`模式，它可以在上面的 git repo 中找到:

```
CREATE TABLE `offset` (
    `group_id` VARCHAR(255),
   `topic` VARCHAR(255),
   `partition` INT,
   `offset` BIGINT,
   PRIMARY KEY (`group_id`, `topic`, `partition`)
);
```

这是`offset`表，将为消费者组的单个主题分区保存和检索偏移量。

要以事务方式使用消息，可以为使用者设置以下配置:

```
 // transaction properties.
        props.put(ConsumerConfig.ISOLATION_LEVEL_CONFIG, "read_committed");
        props.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, "false");
        props.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");
```

*   `ConsumerConfig.ISOLATION_LEVEL_CONFIG`必须是`read_committed`，这意味着只有提交的消息才会被消费者消费。`ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG`必须是`false`，这意味着消费者将手动控制偏移提交。
*   `ConsumerConfig.AUTO_OFFSET_RESET_CONFIG`应为`earliest`，表示如果指定的消费者组 id 的消费者所消费的分区没有 offset commit，则消费者将从分区中的第一个 offset 开始消费消息。

现在，kafka 消费者将使用这些交易配置来消费消息:

```
**try** {
    *// consumer subscribe with consumer rebalance listener.* **consumer**.subscribe(Arrays.*asList*(**topic**), **new** TransactionalConsumerRebalanceListener(**this**));
    **consumer**.poll(0);

    *// When the consumer first starts, after we subscribed to topics, we call poll()
    // once to make sure we join a consumer group and get assigned partitions and
    // then we immediately seek() to the correct offset in the partitions we are assigned
    // to. Keep in mind that seek() only updates the position we are consuming from,
    // so the next poll() will fetch the right messages.* **for** (TopicPartition topicPartition : **this**.**consumer**.assignment()) {
        **long** offset = getOffsetFromDB(**groupId**, topicPartition);
        **consumer**.seek(topicPartition, offset);
        *log*.info(**"consumer seek to the offset [{}] with groupId [{}], topic [{}] and parition [{}]"**, Arrays.*asList*(offset, **groupId**, topicPartition.topic(), topicPartition.partition()).toArray());
    }

    **while** (**true**) {
        *// if wakeupCalled flag set to true, throw WakeupException to exit, before that flushing message by producer
        // and offsets committed by consumer will occur.* **if** (**this**.**wakeupCalled**) {
            **throw new** WakeupException();
        }

        ConsumerRecords<String, Events> records = **consumer**.poll(100);
        **if**(!records.isEmpty()) {
            **for** (ConsumerRecord<String, Events> record : records) {
                String key = record.key();
                Events events = record.value();

                *log*.info(**"key: ["** + key + **"], events: ["** + events.toString() + **"], topic: ["** + record.topic() + **"], partition: ["** + record.partition() + **"], offset: ["** + record.offset() + **"]"**);

                *// process events.* processEvents(events);

                *// an action involved in this db transaction.

                // NOTE: if consumers run with difference group id, avoid saving duplicated events to db.* saveEventsToDB(events);

                *// another action involved in this db transaction.* saveOffsetsToDB(**groupId**, record.topic(), record.partition(), record.offset());
            }

            commitDBTransaction();
        }
    }

} **catch** (WakeupException e) {

} **finally** {
    commitDBTransaction();
    **this**.**consumer**.close();
}
```

消费者将使用消费者重新平衡监听器订阅主题中的消息:

```
**consumer**.subscribe(Arrays.*asList*(**topic**), **new** TransactionalConsumerRebalanceListener(**this**));
```

在`TransactionalConsumerRebalanceListener`消费者重新平衡监听器中，数据库事务将在重新平衡开始之前提交，在消费者被分配到特定分区后，消费者将从数据库中寻找偏移量:

```
**public class** TransactionalConsumerRebalanceListener<K, V> **implements** ConsumerRebalanceListener {

    **private static** Logger *log* = LoggerFactory.*getLogger*(TransactionalConsumerRebalanceListener.**class**);

    **private** AbstractConsumerHandler<K, V> **consumeHandler**;

    **public** TransactionalConsumerRebalanceListener(AbstractConsumerHandler<K, V> consumeHandler)
    {
        **this**.**consumeHandler** = consumeHandler;
    }

    @Override
    **public void** onPartitionsRevoked(Collection<TopicPartition> collection) {
        *// commit db transaction for saving records and offsets to db.* **this**.**consumeHandler**.commitDBTransaction();
    }

    @Override
    **public void** onPartitionsAssigned(Collection<TopicPartition> topicPartitions) {
        **for**(TopicPartition topicPartition : topicPartitions)
        {
            *// get offset from db and let consumer seek to this offset.* String groupId = **this**.**consumeHandler**.**groupId**;
            **long** offset = **this**.**consumeHandler**.getOffsetFromDB(groupId, topicPartition);
            **this**.**consumeHandler**.getConsumer().seek(topicPartition, offset);

            *log*.info(**"in rebalance listener, consumer seek to the offset [{}] with groupId [{}], topic [{}] and parition [{}]"**, Arrays.*asList*(offset, groupId, topicPartition.topic(), topicPartition.partition()).toArray());
        }
    }
}
```

当使用者重新启动时，使用者将从数据库中查找分区的最后更新的偏移量:

```
**long** offset = getOffsetFromDB(**groupId**, topicPartition);
**consumer**.seek(topicPartition, offset);
```

看一下`saveEventsToDB()`和`saveOffsetsToDB()`，在这里处理的结果和偏置将被保存到同一个外部数据库。最后，所有数据库事务都将通过`commitDBTransaction()`提交。

在 Kafka 中，只进行一次处理并不容易，但是正如本文中提到的，您应该分别考虑生产者端和消费者端的事务，并尝试实现事务处理。

您可以从我的 github repo 中查看以下 wiki 页面，了解运行本文中代码的详细信息:

*   [https://github . com/mykidong/Kafka-transaction-example/wiki/Run-Transactional-Kafka-Producer-and-Consumers](https://github.com/mykidong/kafka-transaction-example/wiki/Run-Transactional-Kafka-Producer-and-Consumers)
*   [https://github . com/mykidong/Kafka-transaction-example/wiki/Scenario](https://github.com/mykidong/kafka-transaction-example/wiki/Scenario)