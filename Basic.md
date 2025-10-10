
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
