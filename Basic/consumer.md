## Consumer Offset কী?

Kafka-তে যখন কোনো consumer (গ্রাহক) কোনো topic থেকে data পড়ে, তখন Kafka সেই consumer কোথায় পর্যন্ত পড়েছে তা offset দিয়ে ট্র্যাক করে রাখে।

 Offset হলো প্রতিটি message-এর জন্য একটি unique number (0, 1, 2, 3, ... ইত্যাদি)।
এটা message-এর position বা index এর মতো কাজ করে।
```
 উদাহরণ:

ধরুন আপনার একটা topic আছে — orders
এতে ৫টি message আছে:

Offset	Message
0	Order #1
1	Order #2
2	Order #3
3	Order #4
4	Order #5

এখন আপনার consumer যদি প্রথম ৩টা message পড়ে,
তাহলে Kafka মনে রাখবে যে consumer-এর offset এখন 2 পর্যন্ত পৌঁছেছে।

 মানে consumer 0, 1, 2 offset পর্যন্ত message consume করেছে।

 Consumer Offset কেন দরকার:
কারণ	ব্যাখ্যা
🔁 Restart এর পর থেকে আবার শুরু করার জন্য	যদি consumer বন্ধ হয়ে যায় বা restart হয়, Kafka জানে শেষ কোন offset পর্যন্ত message পড়া হয়েছিল। ফলে consumer সেখান থেকেই আবার শুরু করতে পারে।
✅ Duplicate এড়ানোর জন্য	Offset ট্র্যাক করে রাখলে একই message বারবার পড়া হয় না।
📊 Monitoring ও Lag Check করার জন্য	Offset থেকে বোঝা যায় consumer কত পিছিয়ে আছে (lag) বা কত message এখনো consume হয়নি।
🧩 Offset কোথায় সংরক্ষণ হয়?

Kafka offset সাধারণত __consumer_offsets নামের একটি internal topic-এ সংরক্ষণ করে।
এই টপিক Kafka নিজেই manage করে।

উদাহরণ:

ধরুন consumer group-এর নাম group1
topic orders
partition 0 এর সর্বশেষ offset হলো 100

consumer যদি offset 95 পর্যন্ত পড়ে থাকে,
তাহলে consumer lag = 100 - 95 = 5 message পিছিয়ে আছে।
```
 সংক্ষেপে বাংলা সংজ্ঞা:

Consumer Offset হলো Kafka-র একটি সংখ্যা, যা বলে দেয় কোনো consumer কোন message পর্যন্ত পড়েছে।
এর মাধ্যমে Kafka consumer-এর অবস্থান ট্র্যাক করে, যাতে পুনরায় শুরু হলে বা ত্রুটি ঘটলে ডেটা হারায় না এবং ডুপ্লিকেট না হয়।
