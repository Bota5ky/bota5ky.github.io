### Java Producer

```java
public class ProducerDemo {

    private static final Logger log = LoggerFactory.getLogger(ProducerDemo.class.getSimpleName());

    public static void main(String[] args) {
        log.info("I am a Kafka Producer");

        // create Producer Properties
        Properties properties = new Properties();
        properties.setProperty(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "127.0.0.1:9092");
        properties.setProperty(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
        properties.setProperty(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());

        // create the Producer
        KafkaProducer<String, String> producer = new KafkaProducer<>(properties);

        // create a producer record
        ProducerRecord<String, String> producerRecord =
            new ProducerRecord<>("demo_java", "hello world");

        // send the data - asynchronous
        producer.send(producerRecord);

        // flush data - synchronous
        producer.flush();

        // flush and close producer
        producer.close();

    }
}
```

### Java Producer Callbacks

```java
producer.send(producerRecord, new Callback() {
    @Override
    public void onCompletion(RecordMetadata metadata, Exception e) {
        // executes every time a record is successfully sent or an exception is thrown
        if (e == null){
            // the record was successfully sent
            log.info("Received new metadata. \n" +
                    "Topic: " + metadata.topic() + "\n" +
                    "Partition: " + metadata.partition() + "\n" +
                    "Offset: " + metadata.offset() + "\n" +
                    "Timestamp: " + metadata.timestamp());
        } else {
            log.error("Error while producing", e);
        }
    }
});
```

**粘性分区**：如果发送速度足够快，几条消息可能作为同一批发送到同一分区

```pro
partitioner.class = class org.apache.kafka.clients.producer.internals.DefaultPartitioner
```

`DefaultPartitioner` 初始化时默认采用 `StickyPartitionCache`

### Java Consumer

```java
String boostrapServers = "127.0.0.1:9092";
String groupId = "my-second-application";
String topic = "demo_java";

// create consumer configs
Properties properties = new Properties();
properties.setProperty(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, boostrapServers);
properties.setProperty(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
properties.setProperty(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
properties.setProperty(ConsumerConfig.GROUP_ID_CONFIG, groupId);
properties.setProperty(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");

// create consumer
KafkaConsumer<String, String> consumer = new KafkaConsumer<>(properties);

// subscribe consumer to our topic(s)
consumer.subscribe(Arrays.asList(topic));

// poll for new data
while(true) {
    log.info("Polling");
    ConsumerRecords<String, String> records =
            consumer.poll(Duration.ofMillis(1000));
    for (ConsumerRecord<String, String> record : records) {
        log.info("Key: " + record.key() + ", Value: " + record.value());
        log.info("Partition: " + record.partition() + ", Offset: " + record.offset());
    }
}
```

### Kafka Consumer - Graceful shutdown

```java
// create consumer
KafkaConsumer<String, String> consumer = new KafkaConsumer<>(properties);
// get a reference to the current thread
final Thread mainThread = Thread.currentThread();
// adding the shutdown hook
Runtime.getRuntime().addShutdownHook(new Thread() {
    public void run() {
        log.info("Detected a shutdown, let's exit by calling consumer.wakeup()...");
        consumer.wakeup();
        // 会让下面的poll() throw exception
        // join the main thread to allow the execution of the code in the main thread
        try {
            mainThread.join();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
});
try {
    // subscribe consumer to our topic(s)
    consumer.subscribe(Arrays.asList(topic));
    // poll for new data
    while(true) {
        ConsumerRecords<String, String> records =
                consumer.poll(Duration.ofMillis(1000));
        for (ConsumerRecord<String, String> record : records) {
            log.info("Key: " + record.key() + ", Value: " + record.value());
            log.info("Partition: " + record.partition() + ", Offset: " + record.offset());
        }
    }
} catch (WakeupException e) {
    log.info("Wake up exception!");
    // we ignore this as this is an expected exception when closing a consumer
} catch (Exception e){
    log.error("Unexpected exception");
} finally {
    consumer.close(); // this will also commit the offsets if need be
    log.info("The consumer is now gracefully closed");
}
```

### 自动提交-至多一次

```java
props.put("enable.auto.commit", "true");
props.put("auto.commit.interval.ms", "1000");

consumer.poll(100);
```

### 手动提交-至少一次

```java
props.put("enable.auto.commit", "false");

consumer.commitSync();
```

### 手动指定消费分区和消费位置

#### 指定消费分区

```java
String topic ="foo";
TopicPartition partition0 = new TopicPartition(topic, 0);
TopicPartition partition1 =new TopicPartition(topic,1);
consumer.assign(Arrays.asList(partition0,partition1));
```

#### 指定消费位置

```java
seek(TopicPartition, long)
```