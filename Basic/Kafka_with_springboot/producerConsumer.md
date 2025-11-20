# when we send data kafka
producer: Object daka>>> json>> byte>>kafka
consumer:byte data>>Json>>Json>>object
```

@Service
public class KafkaMessagePublisher {
    @Autowired
    KafkaTemplate<String, UserEventRecord>kafkaTemplate;

    public void sendMessageToKafkaTopic(UserEventRecord  userEvent)
    {
        CompletableFuture<SendResult<String, UserEventRecord>> future = kafkaTemplate
                .send("user-event", userEvent);
        future.whenComplete((result,error)->{
    if (error ==null)
    {

        System.out.println("message consume from consumer  send successfully00000000000"+result.getRecordMetadata().offset()
                );
    }else {

        System.out.println("message not send ");
    }

        });


    }
```
## Record is :
```
package com.sil.kafkaeosb.events;

public record UserEventRecord(String name,
                              String email,String action) {



}
```

}

