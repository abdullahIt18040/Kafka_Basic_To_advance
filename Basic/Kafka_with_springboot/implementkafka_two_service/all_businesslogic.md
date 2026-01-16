## Kafka Topic কী?
```

Topic হলো Kafka-র একটি ডাটা রাখার জায়গা (message stream)
👉 ধরো Topic = একটা খাতা
👉 Message = খাতার প্রতিটা লাইন

২️⃣ Offset কী?

Offset হলো Topic-এর ভিতরে প্রতিটা message-এর unique সিরিয়াল নম্বর

Offset সবসময় 0 থেকে শুরু হয়

নতুন message এলে offset ১ করে বাড়ে

Offset কখনো change হয় না

👉 Offset = লাইনের নম্বর

৩️⃣ Kafka কীভাবে ডাটা store করে?

Kafka ডাটা append-only ভাবে রাখে (শুধু শেষে যোগ হয়)

উদাহরণ 👇
Topic: order-topic

Offset	Message
0	Order Created
1	Payment Done
2	Order Shipped
3	Order Delivered

এখানে,

Order Created → offset = 0

Order Delivered → offset = 3

৪️⃣ Producer কী করে?

Producer message পাঠায় Kafka-তে
👉 Producer offset দেয় না
👉 Kafka নিজেই offset assign করে

Producer → Topic → Kafka assigns offset

৫️⃣ Consumer কীভাবে ডাটা পড়ে?

Consumer offset ব্যবহার করে message পড়ে

Consumer জানে সে কোন offset পর্যন্ত পড়েছে

পরের বার সে last offset + 1 থেকে শুরু করে

উদাহরণ 👇

Consumer last read offset = 1
Next read will start from offset = 2

৬️⃣ Consumer Group & Offset

একটা Consumer Group এর জন্য Kafka আলাদা offset রাখে

Consumer Group	Last Offset
group-A	3
group-B	1

👉 একই topic, কিন্তু আলাদা group হলে আলাদা offset

৭️⃣ Offset কোথায় store হয়?

Kafka offset রাখে একটি internal topic-এ:

__consumer_offsets

৮️⃣ সহজ বাস্তব উদাহরণ

📺 YouTube ভিডিও ভাবো

Video = Topic

Time = Offset

তুমি যেখানে stop করো → Kafka সেই offset মনে রাখে

আবার play করলে সেখান থেকেই শুরু

৯️⃣ কেন Offset গুরুত্বপূর্ণ?

✔ Fault tolerance
✔ Restart হলেও data miss হয় না
✔ Multiple consumer একই topic পড়তে পারে

সংক্ষেপে (One Line):

Kafka-তে message Topic-এর ভিতরে sequential offset সহ store হয়, আর consumer সেই offset ধরে ধরে message পড়ে।
```
