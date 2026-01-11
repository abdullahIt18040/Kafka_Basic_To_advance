## Order Service ↔ Notification Service কীভাবে Kafka দিয়ে communicate করে সেটা সহজ বাংলায়, step-by-step ব্যাখ্যা করছি।
<img width="1127" height="688" alt="image" src="https://github.com/user-attachments/assets/d4fcfa2e-dd6e-4e80-b358-99775b40fdc9" />

```
 Scenario ()

ধরুন আপনার কাছে দুইটা আলাদা Microservice আছে:
Overall Architecture (Kafka-based Microservices)
[ Order Service ]
        |
        |  OrderRecord
        v
Kafka Topic: ordertopic
        |
        v
[ Stock Service ]
        |
        |  OrderPlaceEvent
        v
Kafka Topic: order-place-topic
        |
        v
[ Notification Service ]


👉 এখানে Service ↔ Service direct call নাই
👉 সব communication Kafka event দিয়ে

1️⃣ Order Service (Producer)
📌 কাজ

User order দিলে

Kafka-তে event publish করে

🧾 কোড
@Service
public class KafkaOrderService {

    @Autowired
    private KafkaTemplate<String, OrderRecord> kafkaTemplate;

    public void publishOrderEvent(OrderRecord record) {
        kafkaTemplate.send("ordertopic", record);
    }
}

🔍 কী হচ্ছে?

KafkaTemplate = Kafka producer

OrderRecord = order data (orderId, userId, pid)

Message যাচ্ছে 👉 ordertopic

📤 Example message:

{
  "orderId": 1,
  "userId": 10,
  "pid": 101
}

2️⃣ Stock Service (Consumer + Producer)
📌 কাজ

Order event consume করে

Stock check করে

Result আবার Kafka-তে পাঠায়

🧾 Stock Service কোড
@Service
public class KafkaStockService {

    private static final Map<Integer,Integer> stockMap = Map.of(
        101, 5,
        102, 10
    );

    @KafkaListener(
        topics = {"ordertopic"},
        groupId = "order-consumer-grp"
    )
    @SendTo("order-place-topic")
    public OrderPlaceEvent listenOrder(OrderRecord orderRecord) {

        var PID = orderRecord.pid();
        var stock = stockMap.get(PID);

        if (stock > 0) {
            return new OrderPlaceEvent(
                OrderStatus.SUCCESS,
                orderRecord.userId()
            );
        }

        return new OrderPlaceEvent(
            OrderStatus.FAILED,
            orderRecord.userId()
        );
    }
}

🔍 কী হচ্ছে?

1️⃣ Kafka থেকে OrderRecord আসছে
2️⃣ Stock map থেকে quantity check
3️⃣ Stock থাকলে → SUCCESS
4️⃣ Stock না থাকলে → FAILED
5️⃣ Method যেটা return করছে, সেটা:
```
   ##  @SendTo("order-place-topic")
   @SendTo("order-place-topic") কী?
```
👉 @SendTo ব্যবহার করা হয় KafkaListener method-এর return value আবার Kafka-তে পাঠানোর জন্য।

অর্থাৎ,

Listener = Consumer + Producer একসাথে

🧠 সহজ ভাষায়
@KafkaListener(...)
@SendTo("order-place-topic")
public OrderPlaceEvent listenOrder(OrderRecord orderRecord) {
    return new OrderPlaceEvent(...);
}


এর মানে হলো

1️⃣ Kafka থেকে message consume করো
2️⃣ Business logic চালাও
3️⃣ Method যেটা return করবে
4️⃣ সেটাই আবার Kafka topic-এ পাঠিয়ে দাও

📤 Topic = order-place-topic

```
## how to handle technical issuse and how to handle business issuse error.
## Kafka + Microservices-এ technical issue আর business issue আলাদা করে handle করা best practice
```
Technical Issue vs Business Issue (Concept)
🔴 Technical Issue (System problem)

এগুলো retry করা যায়:

Kafka deserialization error

DB down

Network timeout

Service crash

NullPointerException

Timeout / 5xx error

👉 এগুলো system fix হলে আবার process করা উচিত

🟡 Business Issue (Domain rule failure)

এগুলো retry করলেও লাভ নেই:

Stock নাই

Invalid order

Payment failed (insufficient balance)

User blocked

👉 এগুলো event হিসেবেই publish করতে হব
```
## Core Principle (One-line rule)
<img width="567" height="112" alt="image" src="https://github.com/user-attachments/assets/10ccac7b-7402-4788-aa4a-ef80938af196" />
```
Business error → Event বানাও
Technical error → Retry করো (তারপর DLT)
```
## how to handle error centrally in kafka .

```
Exception কে Event বানিয়ে Kafka Topic-এ পাঠালে কী হয়?
Exception → Event → Kafka Topic

কী ঘটে?

Listener এর ভিতরে exception ধরা (catch) হয়

Exception কে message/event বানানো হয়

অন্য Kafka topic-এ send করা হয়

Listener method successful ভাবে শেষ হয়

Kafka কী ভাবে?

🟢 Kafka বলে: “কাজ শেষ, message consume হয়ে গেছে”

ফলাফল

✅ Offset commit হয়ে যায়

❌ Retry হয় না

❌ @RetryableTopic কাজ করে না

❌ Kafka আর আগের message নিয়ে মাথা ঘামায় না

📌 এটা manual handling, Kafka-র retry mechanism বাইপাস হয়ে যায়

2️⃣ Exception rethrow করলে কী হয়?
Exception → throw → Kafka Retry

কী ঘটে?

Exception ধরা হয় না, অথবা

ধরলেও আবার throw করা হয়

Spring Kafka exception ধরে নেয়

Kafka কী ভাবে?

🔁 Kafka বলে: “এই message fail করেছে, আবার চেষ্টা করবো”

ফলাফল

✅ Retry topic এ পাঠানো হয়

✅ Backoff কাজ করে

✅ Retry শেষ হলে DLT তে যায়

✅ @RetryableTopic কাজ করে

📌 এটা Kafka-র natural & recommended flow
```
## retry and error handling 
<img width="1437" height="863" alt="image" src="https://github.com/user-attachments/assets/01438b88-700e-498c-bdc4-a8b4a400b5a7" />
## my hand note
<img width="670" height="884" alt="image" src="https://github.com/user-attachments/assets/dd9708fd-0250-4227-b59f-8cc371a3c226" />
<img width="475" height="375" alt="image" src="https://github.com/user-attachments/assets/8f11b231-5d46-4d43-914d-3b407f62bd6f" />









