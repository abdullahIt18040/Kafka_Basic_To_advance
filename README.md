## What is Apache Kafka?

Apache Kafka হলো একটি distributed event streaming platform —
মানে এটি এমন একটি সিস্টেম যা real-time data (event/message)
collect, store, process, এবং transfer করতে পারে, দ্রুত এবং নির্ভরযোগ্যভাবে।

তুমি একে এমন ভাবতে পারো যেন Kafka হলো একটি ডেটা পাইপলাইন,
যা এক জায়গা থেকে ডেটা নেয় এবং অন্য জায়গায় পাঠায় — real-time এ।

# Example:

ধরা যাক, একটি বড় ই-কমার্স সাইট আছে (যেমন Amazon):

একজন ইউজার পেমেন্ট করছে 

আরেকজন সার্চ দিচ্ছে 

কেউ আবার পণ্য অর্ডার করছে 

এই সব ঘটনাই (event) একসাথে ঘটছে।
Kafka এই সমস্ত event real-time এ সংগ্রহ করে এবং ভিন্ন ভিন্ন সার্ভিসে পাঠিয়ে দেয় —
যেমন analytics, recommendation system, billing service ইত্যাদি।

##  How Kafka Works (Simple Flow)
Producer → Kafka (Broker/Cluster) → Consumer

Role	কাজ
Producer	মেসেজ বা ডেটা পাঠায় (যেমন order info, logs ইত্যাদি)
Kafka Broker	সেই মেসেজগুলো store করে রাখে (distributed form এ)
Consumer	সেই ডেটা পড়ে নেয় বা process করে (যেমন database, ML system)
# Why We Use Kafka?
কারণ	ব্যাখ্যা
 Real-time Data Processing	Kafka real-time এ data পাঠাতে/গ্রহণ করতে পারে
 High Performance	প্রতি সেকেন্ডে লাখ লাখ মেসেজ হ্যান্ডেল করতে পারে
Scalable	একাধিক server (broker) যোগ করে সহজে বড় করা যায়
 Reliable	মেসেজ হারায় না; replication থাকে
 Decoupling Systems	Producer আর Consumer একে অপরের উপর নির্ভর করে না
Stream Processing Support	Kafka Streams, Flink, Spark এর মাধ্যমে stream process করা যায়
Where Kafka is Used
ক্ষেত্র	ব্যবহার
 E-commerce	Order events, transaction logs
Banking	Fraud detection, transaction tracking
 Analytics	Real-time metrics collection
 Machine Learning	Streaming data pipeline
Social Media	Feed updates, notification system
 In Short:

Kafka = Real-time messaging + Storage + Streaming system

এটি ব্যবহৃত হয় যখন তোমার সিস্টেমে অনেক ডেটা বিভিন্ন জায়গায় real-time এ পাঠানো বা প্রসেস করার দরকার হয়
