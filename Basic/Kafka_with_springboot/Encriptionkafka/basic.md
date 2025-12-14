


package com.sil.kafkaeosb.config;

import com.sil.kafkaeosb.utility.AESUtil;
import jakarta.annotation.PostConstruct;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Configuration;

@Configuration
public class EncryptionConfig {

    @Value("${app.encryption.key}")
    private String encryptionKey;

    @PostConstruct
    public void init() {
        AESUtil.setKey(encryptionKey);
    }
}


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
