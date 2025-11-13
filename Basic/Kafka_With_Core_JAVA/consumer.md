### KafkaConsumer (pure Java client) দিয়ে manual partition assignment করেছো
```

package com.sil.kafkademo.consumerservicce;

import org.apache.kafka.clients.consumer.ConsumerConfig;

import org.apache.kafka.common.TopicPartition;
import org.apache.kafka.common.serialization.StringDeserializer;
import org.springframework.stereotype.Service;
import    org.apache.kafka.clients.consumer.* ;

import java.time.Duration;
import java.util.List;
import java.util.Properties;


public class KafkaConsumer {
    public  static void consumer()
    {
        String TOPIC_NAME = "my-first-topic";
        Properties props = new Properties();
        props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, List.of("localhost:9092"));
        props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG,StringDeserializer.class);
    //    props.put(ConsumerConfig.GROUP_ID_CONFIG,"my-group");
        org.apache.kafka.clients.consumer.KafkaConsumer<String,String> consumer = new org.apache.kafka.clients.consumer.KafkaConsumer<>(props);
//        consumer.subscribe(List.of(TOPIC_NAME,"my-first-topic2"));
       consumer.assign(List.of(new TopicPartition(TOPIC_NAME,0),
                new TopicPartition("Demo2-topic",0),
                new TopicPartition("Demo2-topic",2)
                ));
       consumer.seekToBeginning(
             List.of(new TopicPartition("Demo2-topic",0))
       );
//        ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(1000));

   while (true)
   {
       ConsumerRecords<String, String> records =  consumer.poll(Duration.ofMillis(1000));
     for(var record:records)
     {
         System.out.println("111111111111111111111111111111111111111111111111111");
         System.out.println("Topic is " + record.topic()+" Received message:1111111111111111111111111111111111 " + record.value() +
                 " | partition=" + record.partition() +
                 " | offset=" + record.offset());
     }

     }
   }


    }

when we use subscribe no need to consumer group, but when we used subcribe then we no need to    consumer.assign(List.of(new TopicPartition(TOPIC_NAME,0),
                new TopicPartition("Demo2-topic",0),
                new TopicPartition("Demo2-topic",2)
                ));
       consumer.seekToBeginning(
             List.of(new TopicPartition("Demo2-topic",0))
       );   

       
sfd
```


### Kafka Consumer-এর দুই ধরণের কাজের মোড
```

Kafka-তে consumer data পড়ার জন্য দুইভাবে partition handle করতে পারে:

✅ ১️⃣ subscribe() mode (Automatic partition assignment)

এটা হলো default এবং recommended approach।

🧠 কিভাবে কাজ করে
consumer.subscribe(List.of("my-first-topic", "Demo2-topic"));


➡️ এখানে Kafka broker নিজে সিদ্ধান্ত নেয়,
কোন consumer কোন partition থেকে message পড়বে।

📘 এর মানে:

Kafka consumer group coordination protocol (GroupCoordinator) use করে।

একাধিক consumer থাকলে Kafka load-balance করে (partition-sharing)।

Consumer মারা গেলে Kafka reassign করে অন্য consumer-কে।

Offset স্বয়ংক্রিয়ভাবে track করে group অনুযায়ী।

⚙️ তখন দরকার হয়

GROUP_ID_CONFIG অবশ্যই দিতে হবে ✅
কারণ consumer group না থাকলে broker বুঝবে না কে কে একই group-এ আছে।

props.put(ConsumerConfig.GROUP_ID_CONFIG, "my-group");

🧠 তাহলে subscribe() মোডে নিচের জিনিসগুলো দরকার নেই 👇

assign() ❌ (Kafka নিজেই assign করবে)

seekToBeginning() সাধারণত লাগে না (Kafka offset auto manage করে)

📦 Example
consumer.subscribe(List.of("my-first-topic"));
while (true) {
    ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(1000));
    for (ConsumerRecord<String, String> record : records) {
        System.out.printf("Topic=%s, Partition=%d, Offset=%d, Value=%s%n",
                record.topic(), record.partition(), record.offset(), record.value());
    }
}


🟢 এটা typical, production-grade consumer mode।

🚀 ২️⃣ assign() mode (Manual partition assignment)

এটা হলো manual control mode — তুমি নিজে বলে দাও কোন partition থেকে পড়বে।

🧠 কিভাবে কাজ করে
consumer.assign(List.of(
    new TopicPartition("my-first-topic", 0),
    new TopicPartition("Demo2-topic", 0)
));


➡️ এখানে broker কোনো decision নেয় না,
Kafka group-coordination disable হয়ে যায়।

📘 এর মানে:

Consumer কোনো group coordination করে না।

GROUP_ID_CONFIG লাগবে না।

Offset manually handle করতে হয় (Kafka offset commit করে না)।

তুমি চাইলে seekToBeginning() বা seek(offset) দিয়ে যেখান থেকে পড়বে সেটাও বলে দিতে পারো।

⚙️ Manual control দরকার হয় যখন:

তুমি debugging করছো।

নির্দিষ্ট partition থেকে replay করতে চাও।

custom offset manage করতে চাও।

broker এর auto balancing avoid করতে চাও।

📦 Example
consumer.assign(List.of(new TopicPartition("my-first-topic", 0)));
consumer.seekToBeginning(List.of(new TopicPartition("my-first-topic", 0)));

while (true) {
    ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(1000));
    for (ConsumerRecord<String, String> record : records) {
        System.out.printf("Topic=%s, Partition=%d, Offset=%d, Value=%s%n",
                record.topic(), record.partition(), record.offset(), record.value());
    }
}


4️⃣ Topic এবং Config সেট করা
String TOPIC_NAME = "my-first-topic";
Properties props = new Properties();
props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, List.of("localhost:9092"));
props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
props.put(ConsumerConfig.GROUP_ID_CONFIG, "my-group");

🔍 এখানে কী হচ্ছে:

BOOTSTRAP_SERVERS_CONFIG → Kafka broker কোথায় আছে।

KEY / VALUE_DESERIALIZER_CLASS_CONFIG → message bytes কে String-এ convert করার জন্য।

GROUP_ID_CONFIG → Consumer group-এর নাম (Kafka offset tracking করে group অনুযায়ী)।

⚠️ সমস্যা:
BOOTSTRAP_SERVERS_CONFIG এখানে List.of("localhost:9092") দেওয়া আছে,
কিন্তু এটা আসলে string হওয়া উচিত, Kafka list নিতে পারে না।
✅ সঠিক হবে:

props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");

🧩 5️⃣ Consumer তৈরি করা
org.apache.kafka.clients.consumer.KafkaConsumer<String, String> consumer =
        new org.apache.kafka.clients.consumer.KafkaConsumer<>(props);


➡️ এখানে তুমি সম্পূর্ণ path ব্যবহার করেছো (org.apache.kafka.clients.consumer.KafkaConsumer),
কারণ তোমার class-এর নামও KafkaConsumer, তাই naming clash এড়াতে এটা করা হয়েছে।

🧩 6️⃣ Manual Partition Assignment
consumer.assign(List.of(
    new TopicPartition(TOPIC_NAME, 0),
    new TopicPartition("Demo2-topic", 0),
    new TopicPartition("Demo2-topic", 2)
));


🧠 এই অংশটা সবচেয়ে গুরুত্বপূর্ণ।

এখানে তুমি manual partition assignment করেছো।
মানে — তুমি নিজে Kafka-কে বলে দিচ্ছো কোন topic-এর কোন partition থেকে পড়বে।

🔹 assign() method ব্যবহার করলে Kafka নিজে partition assign করবে না,
তুমি পুরো control নিচ্ছো।

🟠 এজন্য:

Kafka group management disable হয়ে যায়।

Rebalance হয় না।

Offset manually handle করতে হয়।

🧩 7️⃣ Seek to Beginning
consumer.seekToBeginning(List.of(new TopicPartition("Demo2-topic",0)));


👉 এটা Kafka-কে বলে —
“এই partition-এর offset একদম শুরু থেকে পড়া শুরু করো।”
মানে পুরনো message-ও পড়বে।

তুমি চাইলে consumer.seek(topicPartition, specificOffset) ব্যবহার করে নির্দিষ্ট offset থেকেও শুরু করতে পারো।

🧩 8️⃣ Polling Loop
while (true) {
    ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(1000));
    for (var record : records) {
        System.out.println("Topic is " + record.topic()
                + " | Received message: " + record.value()
                + " | partition=" + record.partition()
                + " | offset=" + record.offset());
    }
}


➡️ এই অংশে consumer প্রতি ১ সেকেন্ড পর পর broker থেকে নতুন message নিয়ে আসে।
poll() method মানে — “Kafka broker থেকে message fetch করা।”

🧩 9️⃣ Summary Table
বিষয়	মানে
Consumer type	Pure Java Kafka consumer
Partition assignment	Manual (assign() ব্যবহার করা হয়েছে)
Group ID	আছে, কিন্তু manual mode-এ কাজ করবে না
Auto offset management	বন্ধ (তুমি নিজে offset track করবে যদি দরকার হয়)
Seek ব্যবহার	একদম শুরু থেকে পড়া শুরু করার জন্য
Loop	Infinite, যতক্ষণ না consumer বন্ধ করো
```
