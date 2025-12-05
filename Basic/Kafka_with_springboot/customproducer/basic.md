## custom producer

```
    @Bean

    public KafkaTemplate<String, OrderRecord> orderEventkafkaTemplate() {

        KafkaTemplate<String, OrderRecord> orderRecordkafkaTemplate = new KafkaTemplate(
                kafkaProducerFactoryForOrderEvent());


        return orderRecordkafkaTemplate;
    }
    @Bean
    public DefaultKafkaProducerFactory<String, OrderRecord> kafkaProducerFactoryForOrderEvent() {
        Map<String, Object> properties = new HashMap<>();
        properties.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, List.of("localhost:9092"));
        properties.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        properties.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, OrderEventSerializer.class);

        DefaultKafkaProducerFactory<String, OrderRecord> factory = new DefaultKafkaProducerFactory(properties);


        return factory;
    }
```
