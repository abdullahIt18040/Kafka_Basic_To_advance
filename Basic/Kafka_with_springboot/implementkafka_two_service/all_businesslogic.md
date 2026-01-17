## Kafka Topic কী?
```

Topic হলো Kafka-র একটি ডাটা রাখার জায়গা (message stream)
👉 ধরো Topic = একটা খাতা
👉 Message = খাতার প্রতিটা লাইন

২️⃣ Offset কী?

Offset হলো Topic-এর ভিতরে প্রতিটা message-এর unique সিরিয়াল নম্বর

Offset সবসময় 0 থেকে শুরু হয়

নতুন message এলে offset ১ করে বাড়ে

Offset কখনো change হয় না

👉 Offset = লাইনের নম্বর

৩️⃣ Kafka কীভাবে ডাটা store করে?

Kafka ডাটা append-only ভাবে রাখে (শুধু শেষে যোগ হয়)

উদাহরণ 👇
Topic: order-topic

Offset	Message
0	Order Created
1	Payment Done
2	Order Shipped
3	Order Delivered

এখানে,

Order Created → offset = 0

Order Delivered → offset = 3

৪️⃣ Producer কী করে?

Producer message পাঠায় Kafka-তে
👉 Producer offset দেয় না
👉 Kafka নিজেই offset assign করে

Producer → Topic → Kafka assigns offset

৫️⃣ Consumer কীভাবে ডাটা পড়ে?

Consumer offset ব্যবহার করে message পড়ে

Consumer জানে সে কোন offset পর্যন্ত পড়েছে

পরের বার সে last offset + 1 থেকে শুরু করে

উদাহরণ 👇

Consumer last read offset = 1
Next read will start from offset = 2

৬️⃣ Consumer Group & Offset

একটা Consumer Group এর জন্য Kafka আলাদা offset রাখে

Consumer Group	Last Offset
group-A	3
group-B	1

👉 একই topic, কিন্তু আলাদা group হলে আলাদা offset

৭️⃣ Offset কোথায় store হয়?

Kafka offset রাখে একটি internal topic-এ:

__consumer_offsets

৮️⃣ সহজ বাস্তব উদাহরণ

📺 YouTube ভিডিও ভাবো

Video = Topic

Time = Offset

তুমি যেখানে stop করো → Kafka সেই offset মনে রাখে

আবার play করলে সেখান থেকেই শুরু

৯️⃣ কেন Offset গুরুত্বপূর্ণ?

✔ Fault tolerance
✔ Restart হলেও data miss হয় না
✔ Multiple consumer একই topic পড়তে পারে

সংক্ষেপে (One Line):

Kafka-তে message Topic-এর ভিতরে sequential offset সহ store হয়, আর consumer সেই offset ধরে ধরে message পড়ে।
```
##  Partition কী?
```

Partition হলো Kafka Topic-এর ভিতরের ছোট ছোট ভাগ

👉 Topic = বড় বাক্স
👉 Partition = বাক্সের ভিতরের আলাদা আলাদা খোপ

Kafka ডাটা সরাসরি Topic-এ না রেখে Partition-এ store করে

২️⃣ কেন Partition দরকার?

Partition ব্যবহারের মূল কারণ 👇

✔ High performance (parallel processing)
✔ Scalability (বেশি data handle করা)
✔ Multiple consumer একসাথে কাজ করতে পারে

৩️⃣ Topic → Partition → Message → Offset

একটা Topic-এর ভিতরে একাধিক Partition থাকতে পারে

উদাহরণ 👇
Topic: order-topic
Partitions: 3 (P0, P1, P2)

order-topic
 ├── Partition 0
 ├── Partition 1
 └── Partition 2

৪️⃣ Partition-এর ভিতরে Offset

⚠️ Offset সবসময় Partition অনুযায়ী হয়, Topic অনুযায়ী নয়

Partition 0:

Offset	Message
0	Order-1
1	Order-4

Partition 1:

Offset	Message
0	Order-2
1	Order-5

Partition 2:

Offset	Message
0	Order-3

👉 এখানে দেখো, প্রতিটা Partition-এর offset আলাদা করে 0 থেকে শুরু হয়েছে

৫️⃣ Producer কীভাবে Partition নির্বাচন করে?

Producer message পাঠানোর সময় Partition বেছে নেয় ৩ভাবে 👇

✅ ১. Key দিয়ে
producer.send("order-topic", "userId-101", message);


একই key → সবসময় একই Partition

Order guarantee থাকে

✅ ২. Round Robin (key না দিলে)

একেক message একেক Partition-এ যায়

✅ ৩. Custom Partitioner

নিজের logic লিখে Partition select করা যায়

৬️⃣ Consumer & Partition সম্পর্ক

⚠️ একটা Partition একসময় একটাই Consumer পড়তে পারে (same group-এ)

উদাহরণ 👇

Topic → 3 partitions

Consumer Group → 3 consumers

Consumer-1 → Partition-0
Consumer-2 → Partition-1
Consumer-3 → Partition-2


👉 Consumer বাড়ালে processing speed বাড়ে

৭️⃣ Consumer বেশি হলে কী হবে?

Partition = 3
Consumer = 5

👉 2টা Consumer idle থাকবে
কারণ:

1 Partition = 1 Consumer (per group)

৮️⃣ Order Guarantee কীভাবে হয়?

👉 একই Partition-এর ভিতরে order always maintained
👉 ভিন্ন Partition-এর মধ্যে order guarantee নেই

৯️⃣ Partition বাস্তব উদাহরণ

🛒 E-commerce example:

Topic: order-events

Key: orderId

orderId=101 → Partition-1
orderId=102 → Partition-0
orderId=101 → Partition-1 (same!)


👉 একই order-এর event কখনো mix হবে না

🔟 সংক্ষেপে (One Line)

Partition হলো Kafka Topic-এর ভিতরের parallel data stream, যেখানে প্রতিটা Partition নিজের offset ধরে message store করে।
```
##  Offset Commit মানে কী?
```
Offset commit মানে হলো
👉 Consumer Kafka-কে জানায়:

“আমি এই offset পর্যন্ত message process করেছি”

এতে consumer restart হলেও Kafka জানে কোথা থেকে আবার পড়া শুরু করবে

২️⃣ commitAsync কী?

commitAsync() হলো non-blocking offset commit method

Consumer offset Kafka-তে পাঠায়

wait করে না

পরের message process করতে থাকে

👉 Faster কিন্তু একটু risk আছে

৩️⃣ commitAsync কীভাবে কাজ করে?

ধাপে ধাপে 👇

1. Consumer message পড়ে
2. Business logic process করে
3. commitAsync() কল করে
4. Kafka background-এ offset save করে
5. Consumer next message process করে

৪️⃣ Example (Java)
try {
    while (true) {
        ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(100));

        for (ConsumerRecord<String, String> record : records) {
            process(record);
        }

        consumer.commitAsync();
    }
} finally {
    consumer.close();
}

৫️⃣ commitAsync-এর বৈশিষ্ট্য

✅ Fast
✅ Throughput বেশি
❌ Commit failure হলে consumer জানতেও পারে না
❌ Offset miss হতে পারে (rare case)
```
##  Rebalancing কী?
```

Rebalancing হলো এমন একটি process যেখানে Kafka
👉 Partition গুলো consumer-দের মধ্যে নতুন করে ভাগ করে দেয়

এটা ঘটে যখন Consumer Group-এর পরিবর্তন হয়

২️⃣ কখন Rebalancing হয়?

Rebalancing সাধারণত হয় যখন 👇

✅ ১. নতুন Consumer যোগ হয়
Consumer-1, Consumer-2  →  Consumer-3 join

✅ ২. কোনো Consumer down / crash করে
Consumer-2 died

✅ ৩. Topic-এর Partition সংখ্যা বাড়ে
Partitions: 3 → 5

✅ ৪. Consumer বেশি সময় poll না করে
max.poll.interval.ms exceeded

৩️⃣ Rebalancing-এর সময় কী হয়?

⚠️ এই সময় message processing সাময়িকভাবে থেমে যায়

Step-by-step 👇

1. Kafka সব consumer-কে stop করতে বলে
2. Old partition assignment revoke হয়
3. New partition assignment হয়
4. Consumer নতুন partition থেকে read শুরু করে

৪️⃣ Rebalancing কেন দরকার?

✔ Load evenly distribute করতে
✔ Fault tolerance নিশ্চিত করতে
✔ Scaling সহজ করতে

৫️⃣ Example (সহজভাবে)

Topic: order-topic
Partitions: 3
Consumer Group: order-group

Before Rebalance
Consumer-1 → P0, P1
Consumer-2 → P2

New Consumer Join করল
Consumer-3 join

After Rebalance
Consumer-1 → P0
Consumer-2 → P1
Consumer-3 → P2

৬️⃣ Rebalancing-এর সমস্যা (Side Effects)

❌ Temporary downtime
❌ Duplicate message হতে পারে
❌ Offset commit delay হতে পারে

৭️⃣ Rebalancing কমানোর উপায়
✅ ১. Static Membership ব্যবহার করো
group.instance.id=consumer-1

✅ ২. max.poll.interval.ms ঠিক করো
max.poll.interval.ms=300000

✅ ৩. session.timeout.ms & heartbeat ঠিক রাখো
session.timeout.ms=10000
heartbeat.interval.ms=3000

৮️⃣ Cooperative Rebalancing কী?

Kafka-এর নতুন feature (Incremental Rebalance)

সব consumer stop করে না

ধাপে ধাপে partition move করে

Downtime কম হয়

partition.assignment.strategy=org.apache.kafka.clients.consumer.CooperativeStickyAssignor

৯️⃣ Rebalancing & Offset Commit সম্পর্ক

⚠️ Rebalance হওয়ার আগে offset commit না হলে
👉 Duplicate processing হতে পারে

Best practice 👇

onPartitionsRevoked() {
   commitSync();
}

🔟 One Line Summary

Rebalancing হলো Kafka process যেখানে consumer group পরিবর্তন হলে partition গুলো নতুন করে consumer-দের মধ্যে ভাগ করা হয়।
```
## concurency 3  means:consumer ,for a topic have  3 partion its has  3 consumer (in a consumer group) .3 consumer consumer data from 3 pationtion diffrent threads.
## Kafka-তে Concurrency কীভাবে কাজ করে?
```

Kafka-র rule খুব simple 👇

একই Consumer Group-এ
1 Partition → 1 Consumer Thread

৩️⃣ Example (সহজভাবে)

Topic: order-topic
Partitions: 3

Case 1: 1 Consumer
Consumer-1 → P0, P1, P2


👉 No concurrency (serial processing)

Case 2: 3 Consumers
Consumer-1 → P0
Consumer-2 → P1
Consumer-3 → P2


👉 3x concurrency

Case 3: 5 Consumers
Consumer-1 → P0
Consumer-2 → P1
Consumer-3 → P2
Consumer-4 → Idle
Consumer-5 → Idle


👉 Extra consumer idle থাকবে

৪️⃣ Consumer Thread vs Instance

Concurrency ২ভাবে করা যায় 👇

✅ ১. Multiple Consumer Instance

আলাদা JVM / Pod

Best for production

✅ ২. Multiple Threads (same JVM)

Spring Kafka-তে common

concurrency=3

৫️⃣ Spring Boot Example
@KafkaListener(
    topics = "order-topic",
    groupId = "order-group",
    concurrency = "3"
)
public void consume(String message) {
    process(message);
}


👉 Kafka internally 3টা consumer thread তৈরি করবে.
```
## Offset Commit কী?
```
Offset commit মানে হলো
👉 Consumer Kafka-কে জানায়:

“আমি এই offset পর্যন্ত message process করেছি”

এতে consumer restart হলেও সঠিক জায়গা থেকে পড়া শুরু হয়।

২️⃣ commitSync (Synchronous Commit)

👉 Consumer অপেক্ষা করে যতক্ষণ না Kafka offset save করার confirmation দেয়

কীভাবে কাজ করে?
process message
→ commitSync()
→ Kafka confirms
→ next message

Example
consumer.commitSync();

সুবিধা

✔ Reliable
✔ Failure immediately জানা যায়
✔ Offset loss হয় না

অসুবিধা

❌ Slow
❌ Throughput কম

Use case

Payment

Financial transaction

Critical data

৩️⃣ commitAsync (Asynchronous Commit)

👉 Consumer অপেক্ষা করে না, background-এ commit request পাঠায়

কীভাবে কাজ করে?
process message
→ commitAsync()
→ continue processing

Example
consumer.commitAsync();

সুবিধা

✔ Fast
✔ High throughput

অসুবিধা

❌ Failure silently ignore হতে পারে
❌ Duplicate processing হতে পারে

Use case

Logs

Metrics

Analytics
```
## consumer.seek() কী?
```
consumer.seek() হলো Kafka Consumer API-এর একটি মেথড, যেটা ব্যবহার করে আপনি নিজে থেকে নির্দিষ্ট offset-এ গিয়ে message পড়া শুরু করতে পারেন।

👉 সাধারণত Kafka consumer যেখানে last committed offset ছিল, সেখান থেকেই পড়া শুরু করে।
👉 কিন্তু seek() ব্যবহার করলে আপনি সেই নিয়ম ভেঙে ইচ্ছেমতো offset সেট করতে পারেন।

🔹 কেন seek() দরকার হয়?

consumer.seek() ব্যবহার করা হয় যখন:

🔁 পুরোনো message আবার পড়তে চান

❌ কোনো নির্দিষ্ট message skip করতে চান

🐞 Debug বা testing করার সময় নির্দিষ্ট offset থেকে পড়তে চান

📦 Custom retry বা reprocessing logic করতে চান

🔹 seek() কীভাবে কাজ করে?
consumer.seek(TopicPartition, offset);


এখানে:

TopicPartition → কোন topic-এর কোন partition

offset → কোন offset থেকে পড়া শুরু হবে

🔹 উদাহরণ (Java Kafka Consumer)
TopicPartition tp = new TopicPartition("order-topic", 0);

// এই partition-এ assign হতে হবে আগে
consumer.assign(List.of(tp));

// offset 10 থেকে পড়া শুরু করবে
consumer.seek(tp, 10L);

ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(100));


📌 এখানে consumer order-topic এর partition 0 থেকে offset 10-এর message পড়া শুরু করবে।

```
<img width="1024" height="477" alt="image" src="https://github.com/user-attachments/assets/0104e742-4604-43dd-a11c-34eaa34cdfcc" />

