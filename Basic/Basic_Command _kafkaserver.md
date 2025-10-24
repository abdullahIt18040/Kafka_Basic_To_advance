
## Kafka server Run step by step 


## Generate a cluster ID first:
```
.\bin\windows\kafka-storage.bat random-uuid

```
This will print something like:

fwVwvLx-RlK2Rb5mA8u3SA

```
কমান্ড:
.\bin\windows\kafka-storage.bat random-uuid
```
### কাজ কী করে:

এই কমান্ডটি Kafka ক্লাস্টারের জন্য একটি ইউনিক আইডি (cluster ID) তৈরি করে।
Kafka-র প্রতিটি সার্ভার (broker) একই ক্লাস্টারের অংশ হতে হলে, তাদের সবার একই cluster ID থাকতে হয়।

আউটপুট কেমন হয়:

উদাহরণস্বরূপ, আপনি কমান্ডটি চালালে এটি এরকম কিছু দেখাবে:

fwVwvLx-RlK2Rb5mA8u3SA


এটাই আপনার cluster ID — এটি এলোমেলোভাবে (randomly) তৈরি হয়, এবং একবার তৈরি হলে আপনি এটি পরের ধাপে ব্যবহার করবেন।

# কেন দরকার:

Kafka এখন KRaft mode (Kafka Raft) ব্যবহার করে যেখানে ZooKeeper আর আলাদা লাগে না।
এই মোডে Kafka নিজেই তার মেটাডাটা পরিচালনা করে, তাই প্রতিটি ক্লাস্টারের জন্য একটি ইউনিক cluster ID প্রয়োজন।

## single-node/broker/ Kafka server চালাতে (Windows-এ লোকালি টেস্ট করার জন্য),

তাহলে নিচের কমান্ডটাই ব্যবহার করো 
```

.\bin\windows\kafka-storage.bat format -t bbQsvOxWTiiObpcndlafzA -c .\config\server.properties --standalone

```
এটাই হলো Kafka-র “storage formatting step” —
অর্থাৎ এখন থেকে Kafka জানে এই সার্ভার কোন cluster-এর অংশ এবং কোথায় data রাখবে।

## Kafka সার্ভার চালাও 
```

.\bin\windows\kafka-server-start.bat .\config\server.properties
```

সফলভাবে চললে তুমি দেখবে এরকম কিছু:
[KafkaServer id=1] started (kafka.server.KafkaServer)
INFO [KafkaServer id=1] Kafka Server started in KRaft mode


এবং Kafka এখন running & ready 
## Step-by-Step: Create a Topic in Kafka (Windows)
 নিশ্চিত হও Kafka সার্ভার চলছে

CMD-এ (একটি আলাদা টার্মিনালে) নিচের মতো করে Kafka সার্ভার চালাও:
```

.\bin\windows\kafka-server-start.bat .\config\server.properties
```

Kafka চালু না থাকলে topic তৈরি হবে না।

নতুন CMD উইন্ডো খোলো

এখন তুমি অন্য একটা টার্মিনালে topic তৈরি করবে।

## Topic তৈরি করো
```
নিচের কমান্ড চালাও 

.\bin\windows\kafka-topics.bat --create --topic my-first-topic --bootstrap-server localhost:9092 --partitions 1 --replication-factor 1

 কমান্ড ব্যাখ্যা:
Flag	কাজ
--create	নতুন topic তৈরি করবে
--topic	টপিকের নাম — এখানে my-first-topic
--bootstrap-server	Kafka broker এর ঠিকানা (সাধারণত localhost:9092)
--partitions	কতগুলো partition থাকবে (সাধারণত 1)
--replication-factor	কতটি broker এ কপি থাকবে (single node হলে 1)
Verify (চেক করো Topic তৈরি হয়েছে কিনা)
.\bin\windows\kafka-topics.bat --list --bootstrap-server localhost:9092


তুমি দেখবে 

my-first-topic


মানে Topic সফলভাবে তৈরি হয়েছে 

Optional – Topic Details দেখতে চাও?
.\bin\windows\kafka-topics.bat --describe --topic my-first-topic --bootstrap-server localhost:9092

```
এতে partition ও replication সম্পর্কিত বিস্তারিত দেখাবে।
## ISR (In-Sync Replica) কী?

ISR = In-Sync Replicas
মানে হলো — কোনো partition-এর যত replicas আছে, তার মধ্যে যেগুলো leader-এর সাথে পুরোপুরি sync (up-to-date) আছে, সেগুলোই ISR তালিকায় থাকে।

Kafka-তে প্রতিটি partition-এর একটি leader replica থাকে (যেটি write/read পরিচালনা করে),
আর অন্যান্য replicas হলো followers, যারা leader থেকে data copy করে।

 উদাহরণ দিয়ে বোঝাই:

ধরা যাক, তুমি এই কমান্ড চালাও
```

.\bin\windows\kafka-topics.bat --describe --topic my-first-topic --bootstrap-server localhost:9092


তুমি এরকম আউটপুট পাবে:

Topic: my-first-topic  TopicId: xyz123
PartitionCount: 1  ReplicationFactor: 1  Configs: segment.bytes=1073741824
    Topic: my-first-topic  Partition: 0  Leader: 0  Replicas: 0  Isr: 0

 প্রতিটি অংশের অর্থ:
ফিল্ড	মানে
Topic	টপিকের নাম
Partition	পার্টিশন নম্বর (এখানে 0)
Leader	কোন broker বর্তমানে leader
Replicas	এই partition-এর সব replicas কোন কোন broker-এ আছে
ISR (In-Sync Replica)	যে replicas বর্তমানে leader-এর সাথে sync আছে
 উদাহরণ (multi-broker setup হলে):
Topic: orders  Partition: 0  Leader: 1  Replicas: 1,2,3  Isr: 1,2


 Leader = 1 → Broker 1 is leader
 Replicas = 1,2,3 → তিনটি broker এ copy আছে
 ISR = 1,2 → Broker 1 ও 2 sync এ আছে, কিন্তু broker 3 পিছিয়ে আছে
```
## ISR কেন গুরুত্বপূর্ণ?

ISR বোঝায় কোন replicas safe এবং up-to-date.
Kafka শুধুমাত্র ISR-এর মধ্যে থেকে leader নির্বাচন করে যাতে data loss না হয়।

 এক লাইনে:

ISR (In-Sync Replica) = যে replicas বর্তমানে leader-এর সাথে data sync আছে।

### তুমি যদি Kafka producer-এর সব available options দেখতে চাও (Windows-এ),
তাহলে নিচের কমান্ড চালালেই হবে 
```
 Command:
.\bin\windows\kafka-console-producer.bat --help

 Output (সংক্ষেপে কিছু গুরুত্বপূর্ণ অংশ):

এটা অনেক বড় লিস্ট দেখাবে, কিন্তু নিচে কিছু গুরুত্বপূর্ণ options দেওয়া হলো 

Option	Description
--topic <topic>	কোন টপিকে মেসেজ পাঠাবে তা নির্ধারণ করে
--bootstrap-server <host:port>	Kafka broker address (যেমন localhost:9092)
--property key.separator=:	key এবং value আলাদা করার জন্য separator দেয়
--property parse.key=true	key-value format মেসেজ পাঠাতে সক্ষম করে
--producer.config <file>	custom configuration ফাইল ব্যবহার করতে দেয়
--request-required-acks	producer-এর acknowledgment behavior নির্ধারণ করে
--compression-type	compression enable করে (gzip, snappy, lz4 ইত্যাদি)
```
### Producer চালু করো

এখন সেই একই টপিকে মেসেজ পাঠাতে producer চালাও 
```

.\bin\windows\kafka-console-producer.bat --topic test-topic --bootstrap-server localhost:9092

```
এখন তোমার কনসোল দেখতে এমন হবে:

>Hello Kafka
>This is my first message


এখানে এখন তুমি সরাসরি মেসেজ লিখে Enter চাপলে সেটা টপিকে যাবে।
উদাহরণ:
```
>Hello Kafka
>thi sis abdullah


```
### Consumer চালু করো (অন্য উইন্ডোতে)

একই সময় আরেকটি নতুন টার্মিনাল খুলে নিচের কমান্ড চালাও, যাতে তুমি মেসেজগুলো দেখতে পারো
```

.\bin\windows\kafka-console-consumer.bat --topic test-topic --from-beginning --bootstrap-server localhost:9092

```
তখন তুমি আগের producer থেকে পাঠানো মেসেজগুলো দেখতে পাবে:

Hello Kafka
This is my first message

 সংক্ষিপ্ত সারাংশ:
ধাপ	কমান্ড	কাজ
kafka-server-start.bat .\config\server.properties	সার্ভার চালানো
kafka-topics.bat --create ...	টপিক তৈরি	kafka-console-producer.bat --topic test-topic ...	প্রডিউসার চালানো	kafka-console-consumer.bat --topic test-topic ...	কনজিউমার চালানো

তুমি চাও কি আমি দেখাই কিভাবে producer থেকে JSON

## Help menue 
```

.\bin\windows\kafka-console-consumer.bat -h

```
Kafka will display the help menu, showing all available options you can use with the consumer command. It usually prints something like this:

## Example output you'll see (or similar):
```
Option                                  Description
------                                  -----------
--bootstrap-server <String: server to   REQUIRED: The Kafka server to connect to.
  use for bootstrapping>
--topic <String: topic>                 REQUIRED: The topic id to consume on.
--partition <Integer: partition>        The partition to consume from.
--offset <String: consume offset>       The offset to start consuming from (e.g., earliest, latest).
--from-beginning                        Consume messages from the beginning of the topic.
--group <String: consumer group id>     Consumer group id.
--max-messages <Integer>                Exit after consuming this many messages.
--property <String: name=value>         Kafka consumer properties.
--timeout-ms <Integer: ms>              Timeout for waiting for messages.
--formatter <class>                     Message formatter class.
--help                                  Print usage information (alias: -h).
```
 Most Commonly Used Commands
Task	Command Example
Consume all messages from beginning	

## Add Property 
In Kafka CLI (Command Line), we "add a property" using:
```
--property key=value

```
আপনি যখন কোনও অতিরিক্ত property (যেমন offset, partition ইত্যাদি দেখানোর জন্য) যুক্ত করতে চান, তখন প্রতিটি property আলাদা করে এইভাবে যোগ করতে হয়:
```
--property propertyName=value

 Example: Adding multiple properties

আপনি আগেও যে কমান্ড দিয়েছেন, সেখানে আপনি দুইটি property যোগ করেছেন এভাবে:

--property print.partition=true --property print.offset=true

More Example: যদি আপনি key-ও print করতে চান
.\bin\windows\kafka-console-consumer.bat --bootstrap-server localhost:9092 --topic my-test-topic --property print.partition=true --property print.offset=true --property print.key=true
```
### Full Explain 

Property	কাজ
```
print.partition=true	কোন partition থেকে মেসেজ এসেছে সেটা দেখাবে
print.offset=true	offset (মেসেজের অবস্থান নম্বর) দেখাবে
print.key=true	যদি মেসেজে key থাকে, সেটাও দেখাবে
print.timestamp=true	মেসেজ কবে লেখা হয়েছে তার timestamp দেখাবে
সাধারণ Structure (মনে রাখুন ):
property propertyName=value
```

আপনি যত property চান ততবার --property লিখতে হবে।

### print keys and values

correct command with the right property syntax and spacing:
```
.\bin\windows\kafka-console-producer.bat --topic my-first-topic --bootstrap-server localhost:9092 --property parse.key=true --property key.separator=:


 Explanation:

Part	Meaning
--property parse.key=true	Tells Kafka to treat the message as key-value format.
--property key.separator=:	Defines ":" as the separator between key and value.
 Example usage after running the command:

Now you can type messages in this format:

key1:value1
key2:value2


Each line you enter will be sent to Kafka with:

key = text before :

value = text after :
```
## Multiple Partion Creation 
```
.\bin\windows\kafka-topics.bat --create --topic my-first-topicone --bootstrap-server localhost:9092 --partitions 2 --replication-factor 1
```
## To see Details 
```
.\bin\windows\kafka-topics.bat --describe --topic my-first-topicone --bootstrap-server localhost:9092

```


