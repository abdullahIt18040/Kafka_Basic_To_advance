
### “কেন Kafka-তে একটি Topic-এর মধ্যে multiple partition ব্যবহার করা হয়?”

এটাই Kafka-র performance ও scalability-এর মূল গোপন রহস্য।

চলো সহজ বাংলায় পুরোটা বোঝি 👇
```
🧩 Topic vs Partition

Kafka-তে:

Topic = ডাটা রাখার একটি “লজিক্যাল চ্যানেল”

Partition = ঐ Topic-এর “ছোট ছোট অংশ” বা log segment

অর্থাৎ একটি Topic অনেকগুলো partition-এ ভাগ করা থাকে।

Topic: orders
Partitions: P0, P1, P2, P3

🎯 Multiple Partition ব্যবহারের মূল উদ্দেশ্য
🧠 1. Parallel Processing (একসাথে একাধিক Consumer কাজ করতে পারে)

প্রতিটি partition-এ একসাথে এক বা একাধিক consumer কাজ করতে পারে।
অর্থাৎ data parallel ভাবে consume করা যায়।

👉 উদাহরণ:
ধরা যাক orders topic-এর 3টি partition আছে
আর 3টি consumer একসাথে চলছে।

Partition	Consumer
P0	C1
P1	C2
P2	C3

➡️ এখন প্রত্যেকে আলাদা partition থেকে data নিচ্ছে —
ফলে speed বেড়ে যায় অনেকগুণে।

⚙️ 2. Scalability (বড় system-এ data handle করা সহজ হয়)

একটি broker বা partition একা সব data handle করলে চাপ পড়ে।
কিন্তু partition বাড়ালে data অনেক broker-এ ছড়িয়ে যায়।

ফলে system horizontally scale করা যায় —
broker বাড়ালে partition আরও distribute হয়।

🚀 3. High Throughput (একসাথে অনেক message process করা)

Multiple partition মানে multiple log file।
Producer একাধিক partition-এ একসাথে data লিখতে পারে,
আর Consumers একাধিক partition থেকে একসাথে পড়তে পারে।

ফলে overall throughput অনেক বেড়ে যায় 🔥

🧯 4. Fault Tolerance (data loss থেকে রক্ষা)

প্রতিটি partition replicate করা যায় একাধিক broker-এ।

যেমন:

Partition P0 → Leader (Broker1)
                Follower (Broker2)


যদি Broker1 মারা যায়,
তাহলে Broker2 নতুন Leader হয়ে যায়।

ফলে data loss হয় না ✅

📦 5. Load Balancing (সমানভাবে চাপ ভাগ করা)

Partition বেশি থাকলে data এবং traffic load
বিভিন্ন broker-এর মধ্যে ভাগ হয়ে যায়।

⚖️ সংক্ষেপে তুলনা:
কারণ	লাভ
Parallelism	একাধিক consumer একসাথে কাজ করতে পারে
Scalability	Broker বাড়ালে capacity বাড়ে
High Throughput	বেশি message দ্রুত process হয়
Fault Tolerance	Leader মারা গেলেও Follower কাজ চালায়
Load Balancing	System চাপ সমানভাবে ভাগ হয়
📊 উদাহরণ (চিত্রিত করে ভাবো):
Producer
   ↓
┌────────────┬────────────┬────────────┐
│ Partition0 │ Partition1 │ Partition2 │
│  Broker1   │  Broker2   │  Broker3   │
└────────────┴────────────┴────────────┘
   ↓           ↓            ↓
 Consumer1   Consumer2    Consumer3

``` 
👉 এখন Producer data গুলো 3টি partition-এ ভাগ করে পাঠায়,
আর 3টি consumer একসাথে কাজ করে।
ফলে speed ও performance বেড়ে যায় 3 গুণ পর্যন্ত।

🧠 সহজভাবে মনে রাখো:

“একটা topic-এ multiple partition মানে —
একসাথে অনেক হাত মিলে কাজ করা।” 🖐️🖐️🖐️
কাজ দ্রুত হয়, চাপ ভাগ হয়, আর data নিরাপদ থাকে।

## cluster

A Kafka cluster is a group of one or more Kafka brokers (servers) working together to provide high availability, scalability, and fault tolerance for message streaming. When you hear “Kafka cluster,” think of multiple Kafka servers collaborating to handle data efficiently.
```
✅ Components of a Kafka Cluster
Component	Description
Brokers	These are Kafka servers that store and serve messages. A cluster usually has multiple brokers (e.g., Broker 0, 1, 2).
Zookeeper (Legacy)	Manages cluster metadata (e.g., broker info, topic configs). Required before Kafka v3.0.
Kafka Controller (KRaft mode)	In newer versions (Kafka 3+), KRaft replaces ZooKeeper.
Topics	Logical categories of messages within the cluster.
Partitions	Each topic is split into partitions distributed across brokers.
Producers	Send/write data to the cluster.
Consumers	Read data from the cluster.
✅ How a Kafka Cluster Works (Example)

Let's say you have a topic called orders with 3 partitions:

Partition	Stored On Broker
Partition 0	Broker 1
Partition 1	Broker 2
Partition 2	Broker 3

✅ This distribution allows Kafka to process messages in parallel and gives high throughput.
✅ If one broker fails, replicas on another broker can take over (when replication is enabled).

✅ Why Use a Kafka Cluster?
Advantage	Description
Scalability	Add more brokers to handle more traffic.
Fault Tolerance	Data can be replicated across brokers.
High Availability	Even if one broker fails, others take over.
High Throughput	Multiple partitions allow parallel processing.
✅ Minimum Setup for a Kafka Cluster
Environment	Brokers Needed
Local testing	1 broker (single-node cluster)
Small project	2-3 brokers
Production	3+ brokers (for fault tolerance + replication)
✅ Example: Starting a 3-Broker Cluster Locally (Manually)

Duplicate Kafka config files (server.properties):

server-1.properties, server-2.properties, server-3.properties

Modify:

broker.id=1
listeners=PLAINTEXT://localhost:9093
log.dirs=/tmp/kafka-logs-1


(Repeat for other brokers with different IDs/ports/logs)

Start brokers:

Fault Tolerance মানে হলো — Kafka একাধিক broker-এ ডেটার কপি রেখে দেয়, যাতে কোনো broker নষ্ট হলেও ডেটা হারায় না এবং সিস্টেম বন্ধ না হয়।

```

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
leaderEPOCH how many times election occure  (Vutavuti hoise)
Voters: কোন কোন broker controller হিসেবে ভোটে অংশ নিচ্ছে
HighWatermark: Raft log কতটা পর্যন্ত সিঙ্ক হয়েছে

🧠 সংক্ষেপে বাংলা সংজ্ঞা:

Kafka Quorum হলো Kafka-এর controller নোডগুলোর একটি দল, যারা Raft algorithm ব্যবহার করে cluster-এর metadata ও leader নির্বাচন পরিচালনা করে, যাতে কোনো node নষ্ট হলেও cluster ঠিকভাবে কাজ করতে পারে।
```
## cluser have one leader and other will be flower , flower store or replicate leader data to prevent fault tolarance.
```
১️⃣ Broker Role (ডেটা ম্যানেজার)

👉 Broker হলো Kafka cluster-এর মূল সার্ভার, যেখানে topics ও messages সংরক্ষণ হয়।

⚙️ Broker-এর মূল কাজ:
কাজ	বর্ণনা
💾 Data সংরক্ষণ করা	Producer যে message পাঠায়, Broker তা topic ও partition অনুযায়ী সংরক্ষণ করে।
📤 Data পাঠানো (to consumer)	Consumer যখন ডেটা পড়ে, Broker সেটি পাঠিয়ে দেয়।
⚡ Replication হ্যান্ডল করা	অন্য broker-এ partition-এর কপি (replica) সিঙ্ক রাখে।
🧩 Leader / Follower Partition পরিচালনা	একেকটি partition-এর জন্য এক broker leader থাকে, অন্যরা follower হিসেবে থাকে।
🧩 উদাহরণ:

ধরো তোমার topic orders এর ৩টি partition আছে —

Partition 0 → Broker 1 (Leader)
Partition 1 → Broker 2 (Leader)
Partition 2 → Broker 3 (Leader)


এখন প্রতিটি broker তার partition অনুযায়ী ডেটা রাখে এবং replicate করে।

🧭 ২️⃣ Controller Role (ক্লাস্টার ম্যানেজার)

👉 Controller হলো Kafka-এর “মস্তিষ্ক” 🧠
এটি cluster-এর metadata, broker status, এবং leader election পরিচালনা করে।

⚙️ Controller-এর মূল কাজ:
কাজ	বর্ণনা
🗂️ Cluster metadata পরিচালনা	কোন topic, partition, broker আছে — এইসব তথ্য সংরক্ষণ করে।
🧑‍⚖️ Leader election করা	কোনো broker বা partition leader নষ্ট হলে নতুন leader নির্ধারণ করে।
🛠️ Broker monitoring	কোন broker online/offline আছে তা নজর রাখে।
🔄 Replication coordination	follower partition গুলো leader-এর সাথে sync আছে কি না তা দেখে।
🧩 KRaft Mode-এ Controller কীভাবে কাজ করে:

Kafka 3.x থেকে ZooKeeper বাদ দিয়ে KRaft (Kafka Raft) ব্যবহার শুরু হয়।
এখানে Controller-রা Raft consensus algorithm ব্যবহার করে metadata replicate ও leader নির্বাচন করে।

👉 এই Controller-দের দলকেই বলা হয় Controller Quorum।

⚖️ Broker vs Controller পার্থক্য টেবিলে:
বিষয়	Broker Role	Controller Role
কাজ	Data সংরক্ষণ ও পাঠানো	Metadata ও cluster coordination
অবস্থান	Data plane	Control plane
Port (default)	9092	9093
সংরক্ষিত ডেটা	Topics, partitions, messages	Broker info, topic configs, leaders
Leader/Follower	Partition level	Cluster level
ব্যবহৃত algorithm	Replication protocol	Raft consensus algorithm
প্রকার সংখ্যা	অনেকগুলো broker থাকতে পারে	কয়েকটি controller (সাধারণত 3 বা 5)
📦 সহজভাবে মনে রাখো:

🧱 Broker = ডেটা রাখে ও পাঠায়
🧭 Controller = কে কীভাবে কাজ করবে তা পরিচালনা করে

Broker (ব্রোকার) – ডেটা রাখে ও দেয়
📖 কী এটা:

👉 Broker হলো Kafka-এর একটি server বা node, যা message (data) সংরক্ষণ করে এবং producer ও consumer-এর মধ্যে data আদান-প্রদান করে।

⚙️ কাজ:
কাজ	বর্ণনা
💾 ডেটা সংরক্ষণ	Producer যে message পাঠায়, Broker সেটা topic ও partition অনুযায়ী সংরক্ষণ করে।
📤 Consumer-কে message পাঠানো	Consumer যখন data পড়ে, Broker সেই message তাকে দেয়।
🔁 Replication হ্যান্ডল করা	Partition-এর leader broker তার follower broker-এ data কপি পাঠায়।
🧩 Partition leader হিসেবে কাজ করা	কিছু partition-এর জন্য broker leader হয়, অন্যগুলোর জন্য follower।
📦 উদাহরণ:

ধরো তোমার ৩টা broker আছে 👇

Broker ID	Port	কাজ
1	9092	Leader of partition 0
2	9093	Follower of partition 0
3	9094	Follower of partition 0
🧭 2️⃣ Controller (কন্ট্রোলার) – ক্লাস্টারের মস্তিষ্ক 🧠
📖 কী এটা:

👉 Controller হলো Kafka cluster-এর leader manager,
যে পুরো cluster-এর metadata ও leadership management নিয়ন্ত্রণ করে।

⚙️ কাজ:
কাজ	বর্ণনা
🗂️ Metadata পরিচালনা	কোন topic, partition, broker আছে — এই তথ্য সংরক্ষণ করে।
⚖️ Leader election করে	কোনো broker নষ্ট হলে নতুন leader নির্ধারণ করে।
🧑‍💼 Broker monitor করে	কে online/offline সেটা ট্র্যাক করে।
🔄 Replication coordinate করে	follower broker-গুলো ঠিকমতো leader-এর data sync করছে কিনা তা দেখে।
📦 KRaft Mode-এ:

Kafka 3.x থেকে ZooKeeper বাদ দেওয়া হয়েছে — এখন Controller কাজ করে Raft algorithm দিয়ে।
এখানে Controller-রা একসাথে থাকে একটি Quorum group হিসেবে (যেমন 3 controller node)।

🔁 3️⃣ Follower (ফলোয়ার) – কপিকারক 📦
📖 কী এটা:

👉 Follower হলো সেই broker, যে leader broker-এর data replicate (copy) করে রাখে।
এতে করে data নিরাপদ থাকে এবং fault tolerance পাওয়া যায়।

⚙️ কাজ:
কাজ	বর্ণনা
📥 Leader-এর data কপি করা	Leader partition-এর message follower নিজের কাছে কপি করে রাখে।
🔄 Sync থাকা	Follower সর্বদা leader-এর সাথে synchronized থাকে (latest message অনুযায়ী)।
🛡️ Backup হিসেবে কাজ করা	যদি leader broker নষ্ট হয়, তাহলে follower নতুন leader হতে পারে।
📦 উদাহরণ:

Topic orders এর Partition-0:

Role	Broker ID	Description
Leader	Broker 1	মূল message সংরক্ষণ করছে
Follower	Broker 2	Broker 1-এর data কপি করছে
Follower	Broker 3	Broker 1-এর data কপি করছে

✅ যদি Broker 1 বন্ধ হয়ে যায় → Broker 2 বা 3 নতুন leader হয়ে যায়।

⚖️ তিনটির পার্থক্য একসাথে টেবিলে:
বিষয়	Broker	Controller	Follower
ভূমিকা	ডেটা সংরক্ষণ ও আদানপ্রদান	Cluster metadata ও leader নির্বাচন পরিচালনা	Leader-এর data replicate করে
কাজের ধরন	Data plane	Control plane	Backup plane
Mode	Broker হিসেবে চলে	KRaft বা ZooKeeper mode-এ controller	Broker-এর অংশ
Port	সাধারণত 9092	সাধারণত 9093	Broker-এর মতোই
সংখ্যা	অনেক (৩ বা তার বেশি)	সাধারণত ৩ (quorum group)	প্রতিটি partition-এ ১+
Replication	ডেটা replicate করে	Replication coordinate করে	Replicate করা ডেটা রাখে
Leader election	Controller দ্বারা নির্ধারিত	নিজে election পরিচালনা করে	Leader fail হলে নতুন leader হয়
🧠 সহজভাবে মনে রাখো:

🔸 Broker → ডেটা রাখে ও দেয়
🔸 Controller → কে কী করবে তা ঠিক করে
🔸 Follower → Leader-এর কপি রাখে, প্রয়োজনে তার জায়গা নেয়

Controller (কন্ট্রোলার)

👉 Kafka cluster-এর “manager” বা “মস্তিষ্ক”।

🧠 কাজ:

পুরো cluster metadata (topic, partition, broker info) পরিচালনা করে।

কোনো broker বা partition leader fail করলে নতুন leader নির্বাচন করে।

Broker গুলো online/offline আছে কি না, সেটি মনিটর করে।

📌 Controller পুরো ক্লাস্টারকে manage করে।

👑 2️⃣ Leader (লিডার)

👉 প্রতিটি partition-এর জন্য নির্দিষ্ট একটি broker leader হয়।

⚙️ কাজ:

Producer যে data পাঠায়, সেটা leader partition গ্রহণ করে।

Consumers message পড়লে, সেটাও leader broker থেকে পড়ে।

Leader তার follower-দের কাছে data replicate করে।

📌 Leader শুধু একটি partition-এর data পরিচালনা করে।

⚖️ সংক্ষেপে পার্থক্য:
বিষয়	Controller	Leader
ভূমিকা	Cluster manager	Partition manager
কাজের ক্ষেত্র	পুরো Kafka cluster	নির্দিষ্ট topic partition
সংখ্যা	১টি (বা কয়েকটি controller quorum)	প্রতিটি partition-এর জন্য ১টি
দায়িত্ব	Leader নির্বাচন, broker tracking	Data handle করা
উদাহরণ	Broker 1 Controller হতে পারে	Broker 2 Partition-0-এর Leader হতে পারে

🎯 সহজভাবে মনে রাখো:

🧭 Controller = কে leader হবে তা ঠিক করে
👑 Leader = data handle করে

Kafka Controller সাধারণত data store করে না (topic message data না)
✅ তবে controller node broker হিসেবেও চললে, তখন সে data রাখতে পারে।

চলুন বিস্তারিতভাবে বুঝে নিই 👇

🧭 Kafka Controller-এর কাজ কী?

Controller হলো Kafka cluster-এর “manager” —
এর মূল দায়িত্ব হলো metadata management, leader election, এবং cluster coordination করা।

⚙️ Controller যা করে:
কাজ	ব্যাখ্যা
🗂️ Metadata সংরক্ষণ	কোন topic, partition, broker, leader আছে — এই তথ্য controller-এর কাছে থাকে।
⚖️ Leader নির্বাচন করে	কোনো broker নষ্ট হলে, নতুন leader নির্বাচন করে।
🧑‍💼 Broker monitoring করে	কোন broker online/offline আছে তা দেখে।
🔁 Replication coordinate করে	follower partition-গুলো leader-এর সাথে sync আছে কিনা, তা নিশ্চিত করে।
💾 Controller data রাখে কি?

এখানে “data” বলতে দুই ধরনের বিষয় বোঝানো হয় 👇

ডেটার ধরন	Controller রাখে কি?	ব্যাখ্যা
🧠 Metadata (Topic info, broker list, etc.)	✅ হ্যাঁ	Controller এই তথ্য Raft log-এ রাখে (KRaft mode-এ)।
📦 User Data (messages in topics)	❌ না	এই data broker-এ থাকে, controller-এ না।
🧩 যখন Controller এবং Broker একসাথে হয়:

Kafka 3.x+ (KRaft mode)-এ, তুমি চাইলে একটি node-কে একইসাথে controller + broker হিসেবে চালাতে পারো।

🔸 তখন সেই node দুটি ভূমিকা পালন করবে:

Controller হিসেবে → Metadata পরিচালনা করবে

Broker হিসেবে → Data (topic messages) সংরক্ষণ করবে

উদাহরণ config:
process.roles=broker,controller
node.id=1
controller.listener.names=CONTROLLER
listeners=PLAINTEXT://localhost:9092,CONTROLLER://localhost:9093


👉 এই কনফিগ মানে: একই node controller ও broker দুই কাজই করছে।

⚖️ সংক্ষেপে:
বিষয়	Controller	Broker
Metadata রাখে	✅ হ্যাঁ	⚪ আংশিক
User data (messages) রাখে	❌ না	✅ হ্যাঁ
Role type	Cluster Manager	Data Handler
KRaft Log রাখে	✅ হ্যাঁ	❌ না

🎯 সহজভাবে মনে রাখো:

🧭 Controller data manage করে (metadata)
💾 Broker data store করে (messages)

```

### if we want to create multiple node or server in a cluster we should have create server.property for each node or server.
```
Go 
config>> server.property 
copy it for each node
configure node no, port,log file 

```
### To create multiple node in a cluster we have to write 
 create odd number node in a cluster.  i create  like 3 nodes .
```
C:\kafka>.\bin\windows\kafka-storage.bat format --cluster-id sdlcpro-01521122140 --initial-controllers 111@localhost:9093:dvvIe3AwR4-ycio5syoq4g,222@localhost:9095:0Y_aMQ_aR2i2QshS2cXJyw,333@localhost:9097:Ajw-zSkdTrmUoBS0sh_TwQ --config ./config/server_111.properties
C:\kafka>.\bin\windows\kafka-storage.bat format --cluster-id sdlcpro-01521122140 --initial-controllers 111@localhost:9093:dvvIe3AwR4-ycio5syoq4g,222@localhost:9095:0Y_aMQ_aR2i2QshS2cXJyw,333@localhost:9097:Ajw-zSkdTrmUoBS0sh_TwQ --config ./config/server_222.properties
C:\kafka>.\bin\windows\kafka-storage.bat format --cluster-id sdlcpro-01521122140 --initial-controllers 111@localhost:9093:dvvIe3AwR4-ycio5syoq4g,222@localhost:9095:0Y_aMQ_aR2i2QshS2cXJyw,333@localhost:9097:Ajw-zSkdTrmUoBS0sh_TwQ --config ./config/server_333.properties
```
## Run theses nodes
```
.\bin\windows\kafka-server-start.bat .\config\server_111.properties
.\bin\windows\kafka-server-start.bat .\config\server_222.properties
.\bin\windows\kafka-server-start.bat .\config\server_333.properties

```
## if we want to see  details 
```
C:\kafka>.\bin\windows\kafka-metadata-quorum.bat --bootstrap-controller localhost:9093 describe --status
```
## if we want to see flower or leader details 
```
C:\kafka>.\bin\windows\kafka-metadata-quorum.bat --bootstrap-controller localhost:9093 describe --replication
```


## Create Topic each Nodes
```
.\bin\windows\kafka-topics.bat  --bootstrap-server localhost:9092 --create --topic test

to see list of topic

.\bin\windows\kafka-topics.bat  --bootstrap-server localhost:9092 --list

 to see  describe of topic
.\bin\windows\kafka-topics.bat  --bootstrap-server localhost:9092 --describe --topic test

```
## Create Producer  and consumer 
```
.\bin\windows\kafka-console-producer.bat  --bootstrap-server localhost:9092 --topic test
.\bin\windows\kafka-console-consumer.bat  --bootstrap-server localhost:9092 --topic test --from-beginning
```

## Replication factor 
replication factor must be less then or equal  total node of a cluster
```
.\bin\windows\kafka-topics.bat  --bootstrap-server localhost:9092 --create --topic test2 --replication-factor 3

```
## when multiple nodes are available in  a cluster every 


## Partition leader কে select করে?

👉 Controller node-ই নির্ধারণ করে partition leader কে হবে।
```
⚙️ পুরো প্রক্রিয়া ধাপে ধাপে:
🧩 ধাপ 1: Topic ও Partition তৈরি হয়

যখন তুমি কোনো topic তৈরি করো,
Kafka সেটার partition ও replica গুলো broker গুলিতে ছড়িয়ে দেয়।

উদাহরণ:

Topic: orders
Partitions: 3
Replication Factor: 3


Kafka এগুলো ৩টা broker এ ভাগ করে রাখে।

🧩 ধাপ 2: Controller metadata রাখে

Kafka-এর controller (যে node cluster manage করে) জানে
কোন broker-এ কোন partition-এর copy (replica) আছে।

উদাহরণ:

Partition	Replicas	Current Leader
0	[Broker1, Broker2, Broker3]	Broker1
1	[Broker2, Broker3, Broker1]	Broker2
2	[Broker3, Broker1, Broker2]	Broker3
🧩 ধাপ 3: Controller leader নির্বাচন করে

Controller এই replica list থেকে একজনকে leader বানায়।
অন্যরা follower থাকে।

যদি leader broker down হয়ে যায় —
Controller তা detect করে এবং follower-দের মধ্যে থেকে নতুন leader নির্বাচন করে।

🧩 ধাপ 4: Leader data store করে

Producer যখন data পাঠায় →
Controller producer-কে জানায়, কোন broker সেই partition-এর leader।
Producer কেবল সেই leader broker-এ data পাঠায়।

Leader:

🧾 নতুন data গ্রহণ করে

🔁 follower-দের কাছে replicate করে

📤 consumer-কে data পাঠায়

🧩 Controller কেন নির্বাচন করে?

কারণ Controller-এর কাছে পুরো cluster-এর metadata থাকে —
কে online, কে offline, কোন partition কোন broker-এ আছে — সব তথ্য।

তাই partition leader নির্বাচন Controller-ই করে,
যাতে Kafka cluster সবসময় consistent ও fault-tolerant থাকে।

⚖️ সংক্ষেপে:
কাজ	দায়িত্বে কে
Partition তৈরি করা	Kafka controller
Partition leader নির্বাচন করা	✅ Controller
Partition data store করা	✅ Leader broker
Replication করা	✅ Follower brokers

🎯 সহজভাবে মনে রাখো:
⚙️ 1️⃣ Controller election কে করে?

(এইটা background এ লাগে)

সব controller node একসাথে Raft consensus algorithm ব্যবহার করে একজন Controller Leader নির্বাচন করে।

সেই Controller Leader পুরো cluster পরিচালনা করে।

এরপর সেই Controller Leader-ই partition leader election করায়।

🎯 Controller leader হচ্ছে “Election Manager” — সে ঠিক করে কোন broker partition leader হবে।

⚙️ 2️⃣ Partition leader election (data level)
🧩 কারা অংশ নেয়:

➡️ সেইসব broker, যাদের কাছে partition-এর replica আছে।

🧠 উদাহরণ:

ধরো —

Topic: orders
Partition: 0
Replicas: Broker1, Broker2, Broker3


👉 তাহলে এই তিনটি broker এই partition-এর leader হতে পারে।

```

Leader কে নির্বাচন করে Kafka Controller (Cluster-এর মধ্যে এক বা একাধিক Controller থাকে)।

প্রক্রিয়া:

Controller প্রতিটি Partition-এর জন্য Leader নির্বাচন করে।

নির্বাচন করা হয় ISR (In-Sync Replica) list থেকে —
মানে যারা Leader-এর সাথে data-তে sync আছে।

📦 উদাহরণ:

ধরো, তোমার topic my-topic আছে 3 partition সহ:

Partition	Leader	Followers
P0	Broker 1	Broker 2, Broker 3
P1	Broker 2	Broker 1, Broker 3
P2	Broker 3	Broker 1, Broker 2

👉 যখন producer data পাঠায়:

P0 → Broker 1 (Leader)

P1 → Broker 2 (Leader)

P2 → Broker 3 (Leader)

⚠️ Leader মারা গেলে কী হয়?

যদি Broker 1 মারা যায় (P0-এর leader),
তাহলে Controller সঙ্গে সঙ্গে ISR থেকে নতুন Leader নির্বাচিত করে —
ধরা যাক Broker 2।

নতুন Leader = Broker 2
Follower = Broker 3

এভাবে Kafka data loss ছাড়াই কাজ চালিয়ে যেতে পারে —
এটাই Kafka-র Fault Tolerance।
## more than one partion can assign one node, means 2 or more pation leader can be one node
