```
docker run -d  --name kafka -p 9092:9092              \
-e KAFKA_ADVERTISED_HOST_NAME=kafka                   \
-e KAFKA_ZOOKEEPER_CONNECT=zookeeper:2181             \
-e KAFKA_ADVERTISED_PORT=9092                         \
-e KAFKA_BROKER_ID=1                                  \
-e KAFKA_LISTENERS=PLAINTEXT://:9092                  \
-e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://:9092       \
-e KAFKA_CREATE_TOPICS="stream-in:2:1,stream-out:2:1" \
--link zookeeper  wurstmeister/kafka:1.1.0  
```


```
docker run -d  --name kafka -p 9092:9092              \
-e KAFKA_ADVERTISED_HOST_NAME=kafka                   \
-e KAFKA_ZOOKEEPER_CONNECT=zookeeper:2181             \
-e KAFKA_ADVERTISED_PORT=9092                         \
-e KAFKA_BROKER_ID=1                                  \
-e KAFKA_LISTENERS=PLAINTEXT://:9092                  \
-e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://:9092       \
-e KAFKA_CREATE_TOPICS="stream-in:2:1,stream-out:2:1" \
--link zookeeper  wurstmeister/kafka
```

