## customConsumer

@Configuration
public class CustomConsumerConfig {

    @Bean

    DefaultKafkaConsumerFactory<String, OrderRecord> kafkaOrderfEventConsumerFactory() {
        Map<String, Object> properties = new HashMap<>();
        properties.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG,"localhost:9092");
        properties.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG,StringDeserializer.class);
        properties.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG,OrderEventDeserializer.class);



        DefaultKafkaConsumerFactory<String, OrderRecord> factory = new DefaultKafkaConsumerFactory(properties);

        return factory;
    }
    @Bean
    public ConcurrentKafkaListenerContainerFactory<String,OrderRecord>
    orderEventKafkaListenerContainerFactory()
    {
        ConcurrentKafkaListenerContainerFactory<String,OrderRecord>factory =
                new ConcurrentKafkaListenerContainerFactory<>();
        factory.setConsumerFactory(kafkaOrderfEventConsumerFactory());
        return factory;
    }


}
