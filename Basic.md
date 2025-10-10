
## Kafka server Run step by step 


## Generate a cluster ID first:
```
.\bin\windows\kafka-storage.bat random-uuid

```
This will print something like:

fwVwvLx-RlK2Rb5mA8u3SA

```
কমান্ড:
.\bin\windows\kafka-storage.bat random-uuid
```
### কাজ কী করে:

এই কমান্ডটি Kafka ক্লাস্টারের জন্য একটি ইউনিক আইডি (cluster ID) তৈরি করে।
Kafka-র প্রতিটি সার্ভার (broker) একই ক্লাস্টারের অংশ হতে হলে, তাদের সবার একই cluster ID থাকতে হয়।

আউটপুট কেমন হয়:

উদাহরণস্বরূপ, আপনি কমান্ডটি চালালে এটি এরকম কিছু দেখাবে:

fwVwvLx-RlK2Rb5mA8u3SA


এটাই আপনার cluster ID — এটি এলোমেলোভাবে (randomly) তৈরি হয়, এবং একবার তৈরি হলে আপনি এটি পরের ধাপে ব্যবহার করবেন।

# কেন দরকার:

Kafka এখন KRaft mode (Kafka Raft) ব্যবহার করে যেখানে ZooKeeper আর আলাদা লাগে না।
এই মোডে Kafka নিজেই তার মেটাডাটা পরিচালনা করে, তাই প্রতিটি ক্লাস্টারের জন্য একটি ইউনিক cluster ID প্রয়োজন।

## single-node/broker/ Kafka server চালাতে (Windows-এ লোকালি টেস্ট করার জন্য),

তাহলে নিচের কমান্ডটাই ব্যবহার করো 
```

.\bin\windows\kafka-storage.bat format -t bbQsvOxWTiiObpcndlafzA -c .\config\server.properties --standalone

```
এটাই হলো Kafka-র “storage formatting step” —
অর্থাৎ এখন থেকে Kafka জানে এই সার্ভার কোন cluster-এর অংশ এবং কোথায় data রাখবে।

## Kafka সার্ভার চালাও 
```

.\bin\windows\kafka-server-start.bat .\config\server.properties
```

সফলভাবে চললে তুমি দেখবে এরকম কিছু:
[KafkaServer id=1] started (kafka.server.KafkaServer)
INFO [KafkaServer id=1] Kafka Server started in KRaft mode


এবং Kafka এখন running & ready 
## Step-by-Step: Create a Topic in Kafka (Windows)
 নিশ্চিত হও Kafka সার্ভার চলছে

CMD-এ (একটি আলাদা টার্মিনালে) নিচের মতো করে Kafka সার্ভার চালাও:
```

.\bin\windows\kafka-server-start.bat .\config\server.properties
```

Kafka চালু না থাকলে topic তৈরি হবে না।

নতুন CMD উইন্ডো খোলো

এখন তুমি অন্য একটা টার্মিনালে topic তৈরি করবে।

## Topic তৈরি করো
```
নিচের কমান্ড চালাও 

.\bin\windows\kafka-topics.bat --create --topic my-first-topic --bootstrap-server localhost:9092 --partitions 1 --replication-factor 1

 কমান্ড ব্যাখ্যা:
Flag	কাজ
--create	নতুন topic তৈরি করবে
--topic	টপিকের নাম — এখানে my-first-topic
--bootstrap-server	Kafka broker এর ঠিকানা (সাধারণত localhost:9092)
--partitions	কতগুলো partition থাকবে (সাধারণত 1)
--replication-factor	কতটি broker এ কপি থাকবে (single node হলে 1)
Verify (চেক করো Topic তৈরি হয়েছে কিনা)
.\bin\windows\kafka-topics.bat --list --bootstrap-server localhost:9092
```

তুমি দেখবে 

my-first-topic


মানে Topic সফলভাবে তৈরি হয়েছে 

Optional – Topic Details দেখতে চাও?
.\bin\windows\kafka-topics.bat --describe --topic my-first-topic --bootstrap-server localhost:9092


এতে partition ও replication সম্পর্কিত বিস্তারিত দেখাবে।
