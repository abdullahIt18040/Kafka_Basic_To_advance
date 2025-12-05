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
## how to encrypt + decrypt Kafka messages in Spring Boot using a secret key?
```
public class AESUtil {
    private static String secretKey;
    // called from config to set the key at runtime
    public static void setKey(String key) {
        secretKey = key;
    }
    public static String encrypt(String strToEncrypt) {
        try {
            SecretKeySpec secretKeySpec = new SecretKeySpec(secretKey.getBytes(), "AES");
            Cipher cipher = Cipher.getInstance("AES");
            cipher.init(Cipher.ENCRYPT_MODE, secretKeySpec);
            return Base64.getEncoder().encodeToString(cipher.doFinal(strToEncrypt.getBytes()));
        } catch (Exception e) {
            throw new RuntimeException("Encryption error", e);
        }
    }


    public static String decrypt(String strToDecrypt) {
        try {
            SecretKeySpec secretKeySpec = new SecretKeySpec(
                    secretKey.getBytes(), "AES");
            Cipher cipher = Cipher.getInstance("AES");
            cipher.init(Cipher.DECRYPT_MODE, secretKeySpec);
            byte[] decoded = Base64.getDecoder().decode(strToDecrypt);
            return new String(cipher.doFinal(decoded));
        } catch (Exception e) {
            throw new RuntimeException("Decryption error", e);
        }
    }

}

@Configuration
public class EncryptionConfig {

    @Value("${app.encryption.key}")
    private String encryptionKey;

    @PostConstruct
    public void init() {
        AESUtil.setKey(encryptionKey);
    }
}


```
