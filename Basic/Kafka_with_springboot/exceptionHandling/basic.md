we have to initially enable @EnableKafkaRetryTopic
```
@EnableKafkaRetryTopic
  
@SpringBootApplication
public class KafkaEosbApplication implements CommandLineRunner {
    @Autowired
    private KafkaOrderService kafkaOrderService;
    
    public static void main(String[] args) {
        SpringApplication.run(KafkaEosbApplication.class, args);

    }


    @Override
    public void run(String... args) throws Exception {
    kafkaOrderService.publishOrderEvent(new OrderRecord(123,101,101));
    }
}
```
## Kafka Exception কী?
```

Kafka-তে যখন message পাঠানো (Producer) বা message পড়া (Consumer) সময় কোনো সমস্যা হয়—তখন যেগুলো error হয়, সেগুলোই Kafka exception।

যেমন:

নেটওয়ার্ক সমস্যা

Broker down

ভুল data format

Permission সমস্যা

১️⃣ Kafka Producer Exception Handling
Producer-এ সাধারণ সমস্যা

TimeoutException → সময়ের মধ্যে message পাঠাতে পারেনি

NetworkException → নেটওয়ার্ক সমস্যা

NotEnoughReplicasException → replica পাওয়া যায়নি

SerializationException → object → JSON convert করতে সমস্যা

Producer exception handle করার নিয়ম

✅ Retry ব্যবহার করতে হবে
✅ Callback দিয়ে error ধরতে হবে
✅ প্রয়োজনে DLQ (Dead Letter Queue) এ পাঠাতে হবে

Example (Java Producer)
producer.send(record, (metadata, exception) -> {
    if (exception != null) {
        System.out.println("Kafka message send failed");

        if (exception instanceof RetriableException) {
            // আবার পাঠানো যাবে
        } else {
            // স্থায়ী সমস্যা → DLQ / লগ
        }
    }
});

Producer config (Best Practice)
retries=5
acks=all
enable.idempotence=true

২️⃣ Kafka Consumer Exception Handling
Consumer-এ সাধারণ সমস্যা

CommitFailedException

SerializationException

AuthorizationException

Safe Consumer Structure
try {
    ConsumerRecords<String, String> records =
            consumer.poll(Duration.ofMillis(100));

    for (ConsumerRecord<String, String> record : records) {
        process(record);
    }

    consumer.commitSync();
} catch (Exception e) {
    e.printStackTrace();
} finally {
    consumer.close();
}

৩️⃣ Message Processing Fail হলে কী করবেন?

❌ ভুল পদ্ধতি

Consumer বন্ধ করে দেওয়া

Infinite retry

✅ সঠিক পদ্ধতি

✅ Option 1: Retry + Delay
int retry = 3;
while (retry-- > 0) {
    try {
        process(record);
        break;
    } catch (Exception e) {
        Thread.sleep(1000);
    }
}

✅ Option 2: Dead Letter Queue (DLQ)

যে message বারবার fail করছে, তাকে আলাদা topic-এ পাঠানো।

kafkaTemplate.send("order-dlq", record.key(), record.value());


📌 এতে main consumer চলতে থাকে।

৪️⃣ Spring Boot + Kafka Exception Handling

Spring Kafka ব্যবহার করলে কাজ অনেক সহজ 👌

Global Error Handler
@Bean
public DefaultErrorHandler errorHandler() {
    return new DefaultErrorHandler(
        new DeadLetterPublishingRecoverer(kafkaTemplate),
        new FixedBackOff(1000L, 3)
    );
}


👉 কী হয় এখানে?

৩ বার retry

Fail হলে DLQ তে পাঠাবে

Application বন্ধ হবে না

৫️⃣ Retryable vs Non-Retryable Exception
ধরন	উদাহরণ	কী করবেন
Retryable	Network issue	Retry
Non-Retryable	Serialization error	DLQ
Fatal	Authorization	App বন্ধ
৬️⃣ Production-এ অবশ্যই করবেন

✅ Proper logging
✅ Consumer lag মনিটর
✅ DLQ message সংখ্যা মনিটর
```
<img width="1569" height="909" alt="image" src="https://github.com/user-attachments/assets/eb6b458d-3955-4ef6-b2ab-50150d570a5c" />

hand note is 
<img width="629" height="767" alt="image" src="https://github.com/user-attachments/assets/f9e25b9f-4169-4bb9-a7eb-ae73ee7294de" />
<img width="618" height="559" alt="image" src="https://github.com/user-attachments/assets/e4f96b8d-3a79-4522-9529-35aaab47dcf0" />






