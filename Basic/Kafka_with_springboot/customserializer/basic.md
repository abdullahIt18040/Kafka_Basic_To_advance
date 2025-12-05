## custom serializer

```

public class OrderEventSerializer implements Serializer<OrderRecord> {


    @Override
    public byte[] serialize(String s, OrderRecord orderRecord) {
        StringJoiner joiner = new StringJoiner(":");
        joiner.add(String.valueOf(orderRecord.orderId()));
        joiner.add(String.valueOf(orderRecord.userId()));
      String orderItems = String.join(",", orderRecord.items());
      joiner.add(orderItems);
        String plainText = joiner.toString();

        // Encrypt here
        String encrypted = AESUtil.encrypt(plainText);

        return encrypted.getBytes();
    }

}
```
## deserializer 

```

public class OrderEventDeserializer implements Deserializer<OrderRecord> {


    @Override
    public OrderRecord deserialize(String s, byte[] bytes) {

        String encrypted = new String(bytes);
        // Decrypt here
        String decrypted = AESUtil.decrypt(encrypted);
        // Split fields
        String[] parts = decrypted.split(":");

        Long orderId = Long.parseLong(parts[0]);
        Long userId = Long.parseLong(parts[1]);
        String[] items = parts[2].split(",");
        return new OrderRecord(orderId, userId, List.of(items));
    }
}
```
