## you want more Kafka logs

You can also enable related packages:
```
logging:
  level:
    org.springframework.kafka: TRACE
    org.apache.kafka: TRACE

📌 Common Kafka Log Levels
Level	Use case
TRACE	Very detailed (debugging issues)
DEBUG	Development debugging
INFO	Normal production logs
WARN	Warnings
ERROR	Errors only
⚠️ Note

TRACE logging can generate huge logs, so use it only for debugging.
```
