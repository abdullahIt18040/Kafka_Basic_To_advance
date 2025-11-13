Kafka native client API (KafkaProducer class) ব্যবহার করে ম্যানুয়ালি producer কোড লিখছো

    public static void producer()
    {
        String TOPIC_NAME = "my-first-topic";
        Properties props = new Properties();
        props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, List.of("localhost:9092"));
        props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG,StringSerializer.class);
        // Optional: acknowledgments
        props.put(ProducerConfig.ACKS_CONFIG, "all");

        // Optional: retries
        props.put(ProducerConfig.RETRIES_CONFIG, 3);

        KafkaProducer<String,String> kafkaProducer = new KafkaProducer<>(props);
        ProducerRecord<String,String> record = new ProducerRecord<>(TOPIC_NAME,"HELLO 2 THIS IS SIL from abdullah ");
        kafkaProducer.send(record, new Callback() {
            @Override
            public void onCompletion(RecordMetadata recordMetadata, Exception exception) {
                if (exception == null) {
                    System.out.println("✅ Message sent successfully: "
                            + " | partition=" + recordMetadata.partition()
                            + " | offset=" + recordMetadata.offset());
                } else {
                    System.err.println(" Error sending message: " + exception.getMessage());
                }
            }
        });
    }
