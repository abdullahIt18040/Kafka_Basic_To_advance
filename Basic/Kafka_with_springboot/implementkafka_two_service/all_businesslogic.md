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
