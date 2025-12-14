package com.sil.kafkaeosb.config;

import com.sil.kafkaeosb.events.OrderRecord;
import com.sil.kafkaeosb.serialization.OrderEventDeserializer;
import org.apache.kafka.clients.consumer.ConsumerConfig;
import org.apache.kafka.common.serialization.StringDeserializer;
import org.springframework.beans.factory.ObjectProvider;
import org.springframework.boot.autoconfigure.condition.ConditionalOnMissingBean;
import org.springframework.boot.autoconfigure.kafka.DefaultKafkaConsumerFactoryCustomizer;
import org.springframework.boot.autoconfigure.kafka.KafkaConnectionDetails;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.kafka.config.ConcurrentKafkaListenerContainerFactory;
import org.springframework.kafka.core.ConsumerFactory;
import org.springframework.kafka.core.DefaultKafkaConsumerFactory;

import java.util.HashMap;
import java.util.List;
import java.util.Map;

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


