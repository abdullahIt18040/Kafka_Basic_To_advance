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





