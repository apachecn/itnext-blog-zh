# HowTo:使用 Schema Registry 为 Kafka 生成 Avro 消息

> 原文：<https://itnext.io/howto-produce-avro-messages-to-kafka-ec0b770e1f54?source=collection_archive---------0----------------------->

![](img/074555adb32cf81d6e3f6705e7dd7cba.png)

哈雷戴维森在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上的照片

在 Kafka 中，Avro 是标准的消息格式。

最近我用了[汇流](https://www.confluent.io/product/confluent-open-source/) 3.3.1。我在使用 [Kafka Schema Registry](https://docs.confluent.io/current/schema-registry/docs/index.html) 发送 avro 消息时遇到了一些问题。

在这里，我将向您展示如何使用 Kafka Schema Registry 从客户端应用程序和 Kafka 流发送 avro 消息。

## 向 Kafka 模式注册表注册 Avro 模式

在向主题发送 avro 消息之前，必须向模式注册表注册主题的 avro 模式。

让我们像这样创建 Avro 模式文件 page-view-event.avsc:

```
{
   "type":"record",
   "name":"PageViewEvent",
   "namespace":"com.mykidong.domain.event.schema",
   "fields":[
      {
         "name":"baseProperties",
         "type":[
            "null",
            {
               "type":"record",
               "name":"BaseProperties",
               "namespace":"com.mykidong.domain.event.schema",
               "fields":[
                  {
                     "name":"pcid",
                     "type":[
                        "null",
                        "string"
                     ]
                  },
                  {
                     "name":"sessionId",
                     "type":[
                        "null",
                        "string"
                     ]
                  },
                  {
                     "name":"referer",
                     "type":[
                        "null",
                        "string"
                     ]
                  },
                  {
                     "name":"browser",
                     "type":[
                        "null",
                        "string"
                     ]
                  },
                  {
                     "name":"eventType",
                     "type":[
                        "null",
                        "string"
                     ]
                  },
                  {
                     "name":"version",
                     "type":[
                        "null",
                        "string"
                     ]
                  },
                  {
                     "name":"timestamp",
                     "type":[
                        "null",
                        "long"
                     ]
                  }
               ]
            }
         ]
      },
      {
         "name":"itemId",
         "type":[
            "null",
            "string"
         ]
      },
      {
         "name":"itemTitle",
         "type":[
            "null",
            "string"
         ]
      },
      {
         "name":"scrollRange",
         "type":[
            "null",
            "int"
         ]
      },
      {
         "name":"stayTerm",
         "type":[
            "null",
            "long"
         ]
      },
      {
         "name":"scrollUpDownCount",
         "type":[
            "null",
            "int"
         ]
      }
   ]
}
```

现在，我将使用 Java 库在 [schema registry REST API](https://docs.confluent.io/current/schema-registry/docs/api.html) 中注册 avro schema。

应添加以下汇合 Maven Repo 和依赖项:

```
<repository>
   <id>confluent</id>
   <url>[http://packages.confluent.io/maven/](http://packages.confluent.io/maven/)</url>
</repository><**dependency**>
   <**groupId**>io.confluent</**groupId**>
   <**artifactId**>kafka-schema-registry</**artifactId**>
   <**version**>3.3.1</**version**>
</**dependency**>
<**dependency**>
   <**groupId**>io.confluent</**groupId**>
   <**artifactId**>kafka-avro-serializer</**artifactId**>
   <**version**>3.3.1</**version**>
</**dependency**>
<**dependency**>
   <**groupId**>io.confluent</**groupId**>
   <**artifactId**>kafka-streams-avro-serde</**artifactId**>
   <**version**>3.3.1</**version**>
</**dependency**>
```

在 java 代码中，添加以下内容:

```
// schema registry url.
String url = **"http://broker1:8081"**;// associated topic name.
String topic = "page-view-event";// avro schema avsc file path.
String schemaPath = "/avro-schemas/page-view-event.avsc";// subject convention is "<topic-name>-value"
String subject = topic + **"-value"**;// avsc json string.
String schema = **null**;

FileInputStream inputStream = **new** FileInputStream(schemaPath);
**try** {
    schema = IOUtils.*toString*(inputStream);
} **finally** {
    inputStream.close();
}

Schema avroSchema = **new** Schema.Parser().parse(schema);

CachedSchemaRegistryClient client = **new** CachedSchemaRegistryClient(url, 20);

client.register(subject, avroSchema);
```

您必须看一下主题约定，它是“ <topic-name>-value”，因为我们要注册值模式，而不是键模式。</topic-name>

好了，我们准备发送 avro 消息到主题，页面-视图-事件。

如果不想让 kafka 自动创建主题，可以像这样手动创建:

```
cd $KAFKA_HOME;bin/kafka-topics --create --zookeeper zk1:2181,zk2:2181,zk3:2181 --topic page-view-event --replication-factor 2 --partitions 5 --config cleanup.policy=compact;
```

## 从客户端应用程序生成 Avro 消息

要从客户端应用程序发送 avro 消息，我们必须使用 KafkaProducer API。

让我们为 KafkaProducer 定义 Kafka 配置属性:

```
// kafka broker list.
String brokers = "broker1:9092,broker2:9092";// schema registry url.
String registry = "http://broker1:8081";Properties **producerProps** = **new** Properties();**producerProps**.put(ProducerConfig.***BOOTSTRAP_SERVERS_CONFIG***, brokers);
**producerProps**.put(AbstractKafkaAvroSerDeConfig.***SCHEMA_REGISTRY_URL_CONFIG***, registry);
**producerProps**.put(ProducerConfig.***KEY_SERIALIZER_CLASS_CONFIG***, **"org.apache.kafka.common.serialization.IntegerSerializer"**);
**producerProps**.put(ProducerConfig.***VALUE_SERIALIZER_CLASS_CONFIG***, **"io.confluent.kafka.serializers.KafkaAvroSerializer"**);
**producerProps**.put(ProducerConfig.***ACKS_CONFIG***, **"0"**);
**producerProps**.put(ProducerConfig.***RETRIES_CONFIG***, **"0"**);
**producerProps**.put(ProducerConfig.***BATCH_SIZE_CONFIG***, **"16384"**);
**producerProps**.put(ProducerConfig.***LINGER_MS_CONFIG***, **"1"**);
**producerProps**.put(ProducerConfig.***BUFFER_MEMORY_CONFIG***, **"33554432"**);
```

您必须查看用于序列化 Avro GenericRecords 的值序列化程序，并且必须设置模式注册表 url。

```
// construct kafka producer.
Producer<Integer, GenericRecord> **producer** = **new** KafkaProducer<Integer, GenericRecord>(producerProps);// message key.
int userIdInt = 1;// message value, avro generic record.
GenericRecord record = *buildRecord*();*// send avro message to the topic page-view-event.* **producer**.send(**new** ProducerRecord<Integer, GenericRecord>("**page-view-event"**, userIdInt, record));
```

要构建 Avro GenericRecord，将使用以下内容:

```
**public** GenericRecord buildRecord() { // avro schema avsc file path.
    String schemaPath = "/avro-schemas/page-view-event.avsc"; // avsc json string.
    String schemaString = **null**;

    FileInputStream inputStream = **new** FileInputStream(schemaPath);
    **try** {
        schemaString = IOUtils.*toString*(inputStream);
    } **finally** {
        inputStream.close();
    } // avro schema.
    Schema schema = **new** Schema.Parser().parse(schemaString);    // generic record for page-view-event.
    GenericData.Record record = **new** GenericData.Record(schema); // put the elements according to the avro schema.
    record.put("itemId", "any-item-id"); ...

    **return** record;
}
```

但是您不应该创建如上的 avro 模式实例。因为创建 avro 架构实例需要时间，所以您必须在构建和发送 GenericRecord 之前创建 avro 架构实例。具体见 avro 模式加载器实现:[https://medium . com/@ myki dong/how to-implement-avro-schema-inheritance-757d 2897 C1 ad](https://medium.com/@mykidong/howto-implement-avro-schema-inheritance-757d2897c1ad)

## 从 Kafka 流生成 Avro 消息

从 Kafka Streams 发送 avro，类似于从客户端应用程序发送。区别只是配置属性。

让我们创建 Kafka Streams 配置属性:

```
String appId = "my-app-id";// kafka broker list.
String brokers = "broker1:9092,broker2:9092";// schema registry url.
String registry = "http://broker1:8081";*// serde.* **final** Serde<Integer> integerSerde = Serdes.*Integer*();
**final** Serde<GenericRecord> genericAvroSerde = **new** GenericAvroSerde();

String clientId = appId + **"-client"**;*// stream properties.* Properties **props** = **new** Properties();
**props**.put(StreamsConfig.***APPLICATION_ID_CONFIG***, appId);
**props**.put(StreamsConfig.***CLIENT_ID_CONFIG***, clientId);
**props**.put(StreamsConfig.***BOOTSTRAP_SERVERS_CONFIG***, brokers);
**props**.put(AbstractKafkaAvroSerDeConfig.***SCHEMA_REGISTRY_URL_CONFIG***, registry);
**props**.put(StreamsConfig.***DEFAULT_KEY_SERDE_CLASS_CONFIG***, Serdes.*Integer*().getClass().getName());
**props**.put(StreamsConfig.***DEFAULT_VALUE_SERDE_CLASS_CONFIG***, genericAvroSerde.getClass().getName());
**props**.put(StreamsConfig.*consumerPrefix*(ConsumerConfig.***GROUP_ID_CONFIG***), appId + **"-group"**);
**props**.put(StreamsConfig.*consumerPrefix*(ConsumerConfig.***AUTO_OFFSET_RESET_CONFIG***), **"latest"**);
```

您必须查看默认值 serde 类，它用于序列化来自 Kafka 流的 Avro GenericRecords，这不同于从客户端应用程序发送，并且必须设置 schema registry url。

现在，让我们来看看卡夫卡流代码:

```
**final** Serde<String> stringSerde = Serdes.*String*();KStreamBuilder builder = **new** KStreamBuilder();// read json string from the topic "events".
KStream<Integer, String> jsonStream = builder.stream(integerSerde, stringSerde, "events");KStream<Integer, GenericRecord> pageViewEventStream = jsonStream.map((k, json) -> { // do something, for instance, filter page view event from json and so on.
    ... *// message key.* **int** userIdInt = 1;

    *// message value, avro generic record.* GenericRecord pageViewEventRecord = **buildRecord()**;

    **return new** KeyValue<>(userIdInt, pageViewEventRecord);
});// send page view event avro generic record to the topic "page-view-event".
pageViewEventStream.to(**"page-view-event"**);// start kafka streams.
KafkaStreams **streams** = **new** KafkaStreams(builder, **props**);
**streams**.start();

*// Add shutdown hook to respond to SIGTERM and gracefully close Kafka Streams* Runtime.*getRuntime*().addShutdownHook(**new** Thread(() -> {
    **streams**.close();
}));
```

这个 kafka streams 应用程序首先从主题“events”中读取字符串消息，其中使用了字符串序列化程序。在从 Json 字符串消息中过滤页面视图事件之后，Avro 通用记录被创建并被发送到主题“页面视图事件”。

## 结论

我已经展示了如何使用 kafka Schema Registry 将 avro 通用记录发送到 Kafka，但是您也可以通过修改配置属性中的序列化程序来发送 avro 特定记录。