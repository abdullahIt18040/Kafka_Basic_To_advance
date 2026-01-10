## Order Service ↔ Notification Service কীভাবে Kafka দিয়ে communicate করে সেটা সহজ বাংলায়, step-by-step ব্যাখ্যা করছি।
<img width="1127" height="688" alt="image" src="https://github.com/user-attachments/assets/d4fcfa2e-dd6e-4e80-b358-99775b40fdc9" />

```
🧩 Scenario (বাস্তব উদাহরণ)

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
## how to handle technical issuse and how to handle business issuse error.Kafka + Microservices-এ technical issue আর business issue আলাদা করে handle করা best practice
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


