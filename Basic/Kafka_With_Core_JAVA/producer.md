Kafka native client API (KafkaProducer class) ব্যবহার করে ম্যানুয়ালি producer কোড লিখছো
```
    public static void producer()
    {
        String TOPIC_NAME = "my-first-topic";
        Properties props = new Properties();
        props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, List.of("localhost:9092"));
        props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG,StringSerializer.class);
        // Optional: acknowledgments
        props.put(ProducerConfig.ACKS_CONFIG, "all");

        // Optional: retries
        props.put(ProducerConfig.RETRIES_CONFIG, 3);

        KafkaProducer<String,String> kafkaProducer = new KafkaProducer<>(props);
        ProducerRecord<String,String> record = new ProducerRecord<>(TOPIC_NAME,"HELLO 2 THIS IS SIL from abdullah ");
        kafkaProducer.send(record, new Callback() {
            @Override
            public void onCompletion(RecordMetadata recordMetadata, Exception exception) {
                if (exception == null) {
                    System.out.println("✅ Message sent successfully: "
                            + " | partition=" + recordMetadata.partition()
                            + " | offset=" + recordMetadata.offset());
                } else {
                    System.err.println(" Error sending message: " + exception.getMessage());
                }
            }
        });
    }
```
মেথড ডিক্লারেশন
public static void producer()


এটা একটি static method — সরাসরি ClassName.producer() কল করে ব্যবহার করা যাবে।

এখানে producer logic encapsulate করা হয়েছে।

2️⃣ Topic Name সেট করা
String TOPIC_NAME = "my-first-topic";


Kafka তে যে টপিকে message পাঠাবে তার নাম।

যদি টপিক আগে থেকে না থাকে, broker auto-create করতে পারে (যদি সেটিংস থাকে)।

3️⃣ Properties অবজেক্ট তৈরি
Properties props = new Properties();


Kafka producer এর configuration রাখার জন্য Java Properties object।

এই props এ server address, serializer, acks, retries ইত্যাদি সেট করা হবে।

4️⃣ Bootstrap servers
props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, List.of("localhost:9092"));


Kafka broker এর ঠিকানা যেখানে message পাঠানো হবে।

localhost:9092 মানে broker তোমার লোকাল মেশিনে চলছে।

যদি cluster থাকে, multiple brokers এর address List হিসাবে দিতে পারো।

5️⃣ Serializer সেট করা
props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG,StringSerializer.class);


Kafka message পাঠাতে byte array লাগে।

Key ও Value কে byte array তে convert করার জন্য StringSerializer ব্যবহার করা হয়েছে।

6️⃣ Acknowledgments
props.put(ProducerConfig.ACKS_CONFIG, "all");


Producer কে বলে দেয় broker কত confirmation দেবে।

"all" মানে leader + সব in-sync replicas (ISR) message receive করেছে কিনা check করবে।

বেশি safe, delivery নিশ্চিত হয়।

7️⃣ Retries
props.put(ProducerConfig.RETRIES_CONFIG, 3);


যদি message send করতে গিয়ে network বা broker fail হয়,

producer সর্বোচ্চ 3 বার পুনরায় পাঠানোর চেষ্টা করবে।

8️⃣ KafkaProducer তৈরি
KafkaProducer<String,String> kafkaProducer = new KafkaProducer<>(props);


আসল producer object তৈরি করা হলো।

<String,String> মানে key এবং value দুইটাই String type।

9️⃣ Message (ProducerRecord) তৈরি
ProducerRecord<String,String> record = new ProducerRecord<>(TOPIC_NAME,"HELLO 2 THIS IS SIL from abdullah ");


Topic + value দিয়ে message তৈরি করা হলো।

Key দিলে নির্দিষ্ট partition এ যাবে, না দিলে random partition এ যাবে।

10️⃣ Send message with callback
kafkaProducer.send(record, new Callback() {
    @Override
    public void onCompletion(RecordMetadata recordMetadata, Exception exception) {
        if (exception == null) {
            System.out.println("✅ Message sent successfully: "
                    + " | partition=" + recordMetadata.partition()
                    + " | offset=" + recordMetadata.offset());
        } else {
            System.err.println(" Error sending message: " + exception.getMessage());
        }
    }
});


send() asynchronous (non-blocking) ভাবে message পাঠায়।

Callback দিয়ে message send success বা failure জানতে পারি।

যদি exception null → message সফলভাবে broker-এ write হয়েছে।

RecordMetadata থেকে partition এবং offset info পাওয়া যায়।

Exception হলে error log দেখাবে।

🔹 মনে রাখার বিষয়
```
Producer send asynchronous, তাই callback ব্যবহার করলে পাঠানোর ফলাফল দেখতে পাওয়া যায়।

Acks=all + retries=3 → message delivery খুবই safe।

এখনো producer close করা হয়নি। ভালো practice হলো কাজ শেষে:

kafkaProducer.close();


দেওয়া।

✅ সংক্ষেপে

এই method একটি complete Kafka Producer workflow দেখাচ্ছে:

configuration set করা

KafkaProducer তৈরি করা

message তৈরি করা

send করা asynchronous callback দিয়ে

success/failure লগ করা.
```
## এখানে যদি acks=all থাকে:
```
Broker তখন leader + all in-sync replicas (ISR) থেকে acknowledgment পাবে।

যদি কোনো replica down থাকে, producer exception throw করবে, যা তুমি callback এ দেখতে পারবে।

অর্থাৎ, logs এ কোনো error না আসা মানে message সব replicas-এ safely write হয়েছে।

Acknowledgments (ACKS_CONFIG)
props.put(ProducerConfig.ACKS_CONFIG, "all");

🔹 কী বোঝাচ্ছে

Producer যখন message broker-এ পাঠায়, তখন broker কিভাবে confirmation দিবে সেটি নির্ধারণ করে acks।

Kafka-তে তিনটি option থাকে:

Value	মানে	কিভাবে কাজ করে
0	No acknowledgment	Producer message পাঠাবে কিন্তু কোনো confirmation পাবে না। দ্রুত কিন্তু risky।
1	Leader acknowledgment	শুধু leader broker acknowledgment পাঠাবে। Follower যদি down হয়, message lost হতে পারে।
all (বা -1)	All replicas acknowledgment	Leader + সব ISR (in-sync replicas) message receive করেছে কিনা check করবে। সবচেয়ে safe।
🔹 তুমি কেন "all" ব্যবহার করছো?

যাতে message delivery durable ও consistent হয়।

কোনো replica down হলেও, ISR-এ সব replica message পাওয়ার পর confirmation যাবে।

Safety বেশি, latency একটু বাড়তে পারে।

2️⃣ Retries (RETRIES_CONFIG)
props.put(ProducerConfig.RETRIES_CONFIG, 3);

🔹 কী বোঝাচ্ছে

যদি message পাঠানোর সময় network issue, leader fail বা broker unavailable হয়,

Producer কতোবার পুনরায় পাঠানোর চেষ্টা করবে তা এখানে নির্ধারণ করা হয়।

🔹 এখানে 3 মানে

কোনো message fail হলে producer সর্বোচ্চ 3 বার retry করবে।

Default হলো 0 (retry নেই) → message fail হলে exception যাবে।

🔹 কেন প্রয়োজন

Kafka একটি distributed system, network বা broker failure হতে পারে।

Retry দিলে transient errors-এ message lost হওয়ার ঝুঁকি কমে।
```
