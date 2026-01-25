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
we must implement
<img width="975" height="472" alt="image" src="https://github.com/user-attachments/assets/feb2465e-da79-4e39-9c3b-1e63a0c9e0a6" />

```
<img width="1569" height="909" alt="image" src="https://github.com/user-attachments/assets/eb6b458d-3955-4ef6-b2ab-50150d570a5c" />

hand note is 
<img width="629" height="767" alt="image" src="https://github.com/user-attachments/assets/f9e25b9f-4169-4bb9-a7eb-ae73ee7294de" />
<img width="618" height="559" alt="image" src="https://github.com/user-attachments/assets/e4f96b8d-3a79-4522-9529-35aaab47dcf0" />

## @EnableScheduling কী?
```

@EnableScheduling হলো Spring-এর একটি annotation
এটা দিলে Spring বুঝে নেয় যে—

👉 এই application-এ scheduled (সময় অনুযায়ী) কাজ চলবে

মানে:

নির্দিষ্ট সময় পরপর কাজ চলবে

অথবা নির্দিষ্ট সময়ে একবার কাজ চলবে

🔹 এটা কেন দরকার?

Spring-এ আপনি যদি এই annotation দেন:

@Scheduled(fixedDelay = 5000)
public void doSomething() {
    System.out.println("Hello");
}


❌ কিন্তু @EnableScheduling না দেন → method কখনোই চলবে না

👉 কারণ Spring scheduler enable হয়নি।

🔹 কোথায় ব্যবহার করবেন?

সাধারণত main application class বা config class-এ।

Example
@SpringBootApplication
@EnableScheduling
public class KafkaEosbApplication {
    public static void main(String[] args) {
        SpringApplication.run(KafkaEosbApplication.class, args);
    }
}

🔹 @EnableScheduling কীভাবে কাজ করে? (Simple Flow)

1️⃣ Application start হয়
2️⃣ Spring scheduler activate হয়
3️⃣ @Scheduled দেওয়া method খুঁজে
4️⃣ সময় অনুযায়ী auto execute করে

🔹 @Scheduled এর ধরন
1️⃣ Fixed Rate
@Scheduled(fixedRate = 5000)


⏱ প্রতি ৫ সেকেন্ডে একবার (previous শেষ হোক বা না হোক)

2️⃣ Fixed Delay
@Scheduled(fixedDelay = 5000)


⏳ আগের কাজ শেষ হওয়ার ৫ সেকেন্ড পর আবার শুরু

3️⃣ Cron Expression
@Scheduled(cron = "0 0 2 * * ?")


🕑 প্রতিদিন রাত ২টায়

🔹 Kafka + @EnableScheduling (আপনার Case)

আপনি যখন @RetryableTopic ব্যবহার করছেন:

@RetryableTopic(backoff = @Backoff(delay = 5000))


👉 Spring internally:

delay handle করে

retry schedule করে

📌 এজন্য scheduler দরকার

👉 আপনি যখন TaskScheduler bean দেন বা
@EnableScheduling enable করেন, তখন Spring এই কাজ করতে পারে।

⚠️ Note:
@RetryableTopic এর জন্য TaskScheduler MUST,
@EnableScheduling একা সবসময় যথেষ্ট না — কিন্তু useful।


```



