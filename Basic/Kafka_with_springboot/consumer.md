## consumer
```

@Service
public class kafkaMessageListener {

    @KafkaListener(topics = "user-event",groupId = "my-group21")
    public void consumer21(UserEventRecord data)
    {
        System.out.println("Spring boot consumer data receivce................................................... "+data);
    }
}
```
