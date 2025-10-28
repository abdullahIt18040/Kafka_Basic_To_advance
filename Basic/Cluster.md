## Kafka Cluster কী?

Cluster have Nodes or servers and each node have bootstrap server port like 9092 ,and bootstrap controller  port : like 9093


> bootstrap server port used , connect consumer and producer
> bootstrap controller port used to communication among servers or nodes in cluster


 Kafka Cluster হলো একাধিক Kafka Broker (server)-এর একটি দল, যারা একসাথে কাজ করে message সংরক্ষণ, প্রেরণ ও প্রক্রিয়াকরণের জন্য।

অর্থাৎ, Kafka Cluster = অনেকগুলো Broker মিলে গঠিত একটি বড় সিস্টেম।
```
⚙️ Kafka Cluster-এর মূল অংশগুলো:
অংশ	কাজ
Broker	Kafka-এর মূল সার্ভার, যেখানে message সংরক্ষণ হয়। প্রতিটি broker একটি ID পায় (যেমন broker-1, broker-2, broker-3)।
Topic	ডেটা রাখার জন্য লজিক্যাল নাম বা ক্যাটেগরি (যেমন orders, payments)।
Partition	প্রতিটি topic ছোট ছোট ভাগে ভাগ হয়।
Producer	যে Kafka-তে data পাঠায় (write করে)।
Consumer	যে Kafka থেকে data পড়ে (read করে)।
Controller (ZooKeeper বা KRaft)	Cluster-এর leader নির্বাচন ও broker সমন্বয় করে।
🧩 একটি উদাহরণ দিয়ে বোঝা যাক:

ধরুন, আপনার Kafka Cluster-এ ৩টি broker আছে 👇

Broker	Port	Description
Broker 1	9092	Topic partition-0
Broker 2	9093	Topic partition-1
Broker 3	9094	Topic partition-2

📦 আপনি যদি orders নামের একটি topic তৈরি করেন এবং সেটিতে ৩টি partition দেন —
তাহলে প্রতিটি partition আলাদা broker-এ সংরক্ষিত হবে।

👉 এতে ডেটা distributed হয়ে যায়, এবং load balancing হয়।

🔁 Fault Tolerance (ত্রুটি সহনশীলতা):

Kafka Cluster-এর বড় সুবিধা হলো —
যদি কোনো broker বন্ধ হয়ে যায়, অন্য broker তার replica ব্যবহার করে কাজ চালিয়ে যেতে পারে।
এটাই replication এবং fault tolerance এর মূল উদ্দেশ্য।

🚀 Kafka Cluster-এর সুবিধা:
সুবিধা	ব্যাখ্যা
⚡ Scalability	বেশি লোড সামলাতে নতুন broker যোগ করা যায়।
🛡️ Fault Tolerance	কোনো broker বন্ধ হলে অন্য broker কাজ চালিয়ে নেয়।
📈 High Throughput	একাধিক partition parallel ভাবে message হ্যান্ডল করে।
🔗 Replication	ডেটার কপি বিভিন্ন broker-এ থাকে, ফলে ডেটা হারায় না।
🖥️ Local Cluster (Single-node)

যদি আপনি শুধু নিজের কম্পিউটারে Kafka চালান (একটা broker), তাও সেটা একটা single-node cluster।
Production সিস্টেমে সাধারণত ৩ বা তার বেশি broker থাকে।
```
সংক্ষেপে বাংলা সংজ্ঞা:
Kafka Cluster হলো একাধিক Kafka Broker-এর সমন্বয়ে গঠিত একটি ডিস্ট্রিবিউটেড সিস্টেম, যেখানে ডেটা ভাগ হয়ে সংরক্ষিত হয় এবং কোনো সার্ভার নষ্ট হলেও সিস্টেম কাজ চালিয়ে যেতে পারে।

চমৎকার প্রশ্ন 👍
Kafka-তে “Quorum” (উচ্চারণ: কোরাম) একটি গুরুত্বপূর্ণ ধারণা, বিশেষ করে KRaft mode-এ (Kafka Raft metadata mode)।
চলুন সহজ বাংলায় ব্যাখ্যা করি 👇

 ## Kafka Quorum কী?
```
Quorum মানে হলো —
একটি group বা দল, যারা একসাথে Kafka cluster-এর metadata বা control সিদ্ধান্ত নেয়।

Kafka যখন ZooKeeper বাদ দিয়ে নিজস্ব KRaft (Kafka Raft) সিস্টেমে চলে আসে,
তখন metadata management এই quorum-ভিত্তিক system দ্বারা পরিচালিত হয়।

⚙️ Quorum-এর কাজ কী?

Kafka Cluster-এর মধ্যে কয়েকটি broker থাকে যারা controller হিসেবে কাজ করে।
এরা মিলে একটি Quorum তৈরি করে।

এই quorum-এর কাজ হলো —

কে controller হবে তা নির্ধারণ করা

cluster-এর metadata (topic, partition, broker info ইত্যাদি) সংরক্ষণ করা

failover হলে নতুন leader নির্বাচন করা

🧩 সহজ উদাহরণ:

ধরুন আপনার Kafka cluster-এ ৩টি controller আছে 👇

Controller ID	Role
1	Leader
2	Follower
3	Follower

এরা মিলে একটি Quorum গঠন করে।

যদি leader controller (1) নষ্ট হয়ে যায়, তাহলে quorum-এর বাকি সদস্যরা ভোট দিয়ে (Raft consensus) নতুন leader নির্বাচন করবে।

✅ এর ফলে Kafka চলতে থাকে — সিস্টেম বন্ধ হয় না।

🔁 Quorum কেন দরকার:
কারণ	ব্যাখ্যা
🛡️ Fault Tolerance	কোনো controller নষ্ট হলেও quorum বাকিরা নতুন leader নির্বাচন করে।
📊 Consistency	সব controller একই metadata রাখে, যাতে কোনো অসঙ্গতি না হয়।
⚙️ ZooKeeper-এর বিকল্প	Kafka এখন নিজেই metadata পরিচালনা করে, ZooKeeper ছাড়াই।
⚙️ Quorum কিভাবে দেখা যায় (Windows):

Kafka যদি KRaft mode-এ চলে, তাহলে আপনি নিচের কমান্ড দিয়ে quorum-এর অবস্থা দেখতে পারেন 👇

C:\kafka>.\bin\windows\kafka-metadata-quorum.bat --bootstrap-controller localhost:9093 describe --status

📋 Output (উদাহরণ):
ClusterId: F6Yh3kQ0Tq2wQ
LeaderId: 1
LeaderEpoch: 5
HighWatermark: 20
Voters: [1,2,3]
Observers: []


LeaderId: বর্তমানে কে controller leader
Voters: কোন কোন broker controller হিসেবে ভোটে অংশ নিচ্ছে
HighWatermark: Raft log কতটা পর্যন্ত সিঙ্ক হয়েছে

🧠 সংক্ষেপে বাংলা সংজ্ঞা:

Kafka Quorum হলো Kafka-এর controller নোডগুলোর একটি দল, যারা Raft algorithm ব্যবহার করে cluster-এর metadata ও leader নির্বাচন পরিচালনা করে, যাতে কোনো node নষ্ট হলেও cluster ঠিকভাবে কাজ করতে পারে।

```
