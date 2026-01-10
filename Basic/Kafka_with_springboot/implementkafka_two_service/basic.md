## Order Service ↔ Notification Service কীভাবে Kafka দিয়ে communicate করে সেটা সহজ বাংলায়, step-by-step ব্যাখ্যা করছি।
```
🧩 Scenario (বাস্তব উদাহরণ)

ধরুন আপনার কাছে দুইটা আলাদা Microservice আছে:

1️⃣ Order Service

👉 Order create / update করে
👉 Kafka-তে event পাঠায়

2️⃣ Notification Service

👉 Kafka থেকে event শুনে
👉 Email / SMS / Push Notification পাঠায়

এরা একজন আরেকজনকে সরাসরি call করে না (REST না),
এরা Kafka এর মাধ্যমে কথা বলে।

🔗 Kafka কীভাবে মাঝখানে কাজ করে

Kafka এখানে Message Broker / Event Bus হিসেবে কাজ করে।

Order Service  ──(event)──▶  Kafka Topic  ──(consume)──▶ Notification Service

🧱 Step-by-Step Architecture
┌──────────────┐
│ Order Service│
│ (Producer)   │
└─────┬────────┘
      │  OrderCreatedEvent
      ▼
┌──────────────────┐
│ Kafka Topic       │
│  order-events     │
└─────┬────────────┘
      │
      ▼
┌──────────────────┐
│ Notification     │
│ Service          │
│ (Consumer)       │
└──────────────────┘

🧾 Step 1: Order Service (Producer)

Order তৈরি হলে একটা event object বানানো হয়।

🟢 Event Data (OrderEvent)
{
  "orderId": 101,
  "userId": 55,
  "amount": 500,
  "status": "CREATED"
}

🟢 Order Service কী করে?

Order DB-তে save করে

Kafka topic-এ message পাঠায়

👉 Kafka Producer

kafkaTemplate.send("order-events", orderEvent);


Order Service এখানে জানেই না কে consume করবে।

📦 Step 2: Kafka Topic
Topic Name: order-events

Kafka topic হলো:

Durable (ডাটা হারায় না)

Queue না, log

Multiple consumer পড়তে পারে

Kafka শুধু message store + deliver করে।

🔔 Step 3: Notification Service (Consumer)

Notification Service Kafka-তে listener বসায়।

@KafkaListener(topics = "order-events", groupId = "notification-group")
public void consume(OrderEvent event) {
    sendNotification(event);
}


Kafka যখনই নতুন message পাবে:
➡ Notification Service automatically পেয়ে যাবে।

📨 Notification Service কী করে?

Event পেলে:

Email পাঠায়

SMS পাঠায়

Push Notification পাঠায়

Order Service এসব জানেই না ❌
Loose Coupling ✔
```
