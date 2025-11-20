package com.sil.kafkaeosb.kafkaService;

import com.sil.kafkaeosb.events.UserEventRecord;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.kafka.annotation.KafkaListener;
import org.springframework.stereotype.Service;

@Service
public class kafkaMessageListener {

    @KafkaListener(topics = "user-event",groupId = "my-group21")
    public void consumer21(UserEventRecord data)
    {
        System.out.println("Spring boot consumer data receivce................................................... "+data);
    }





}
