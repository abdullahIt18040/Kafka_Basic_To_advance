## Kafka Cluster কী?

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

