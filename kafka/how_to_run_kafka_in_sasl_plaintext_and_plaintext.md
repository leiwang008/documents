# Config Kafka broker in PLAINTEXT at port 9092 and SASL_PLAINTEXT at port 9093. And config consumer/producer to connect the broker.

My experience [pass config file with correct option for different script](https://stackoverflow.com/questions/78564432/unexpected-kafka-request-of-type-metadata-during-sasl-handshake-when-connecting/78568514#78568514), I just want to share it.

+ --command-config for kafka-topics.bat 
+ --consumer.config for kafka-console-consumer.bat 
+ --producer.config for kafka-console-producer.bat


kafka-topics.bat --list --bootstrap-server localhost:9093 --command-config ./config/client.properties


kafka-console-consumer.bat --topic gaming-events --from-beginning --bootstrap-server localhost:9092
kafka-console-producer.bat --topic gaming-events --bootstrap-server localhost:9092

kafka-console-consumer.bat --topic gaming-events --from-beginning --bootstrap-server localhost:9093 --consumer.config ./config/consumer.properties
kafka-console-producer.bat --topic gaming-events --bootstrap-server localhost:9093 --producer.config ./config/producer.properties
kafka-console-producer.bat --topic gaming-events --bootstrap-server safsdev03.na.sas.com:9093 --producer.config ./config/producer.properties

For more security config, refer to [https://kafka.apache.org/documentation/#security](https://kafka.apache.org/documentation/#security), but I can assure you the official document is not so good.

1. Setup the server to have a PLAINTEXT at port 9092 and SASL_PLAINTEXT at 9093

```bash
listeners=PLAINTEXT://localhost:9092, SASL_PLAINTEXT://localhost:9093
security.inter.broker.protocol=SASL_PLAINTEXT
sasl.mechanism.inter.broker.protocol=PLAIN
sasl.enabled.mechanisms=PLAIN

# we can also specify the sasl config information instead of using the followign cinfig file 'kafka_jaas.conf'
listener.name.sasl_plaintext.plain.sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required \
    username="admin"
    password="admin-secret"
    user_admin="admin-secret"
    user_alice="alice-secret";
```

2. Create a file 'kafka\_jaas.conf' under the foler 'config' with the following content

```bash
sasl_plaintext.KafkaServer{
    org.apache.kafka.common.security.plain.PlainLoginModule required
    username="admin"
    password="admin-secret"
    user_admin="admin-secret"
    user_alice="alice-secret";
};
```

3. Modify the kafka-server-start.bat script to set the JVM parameter 'java.security.auth.login.config' by environment JAAS\_OPTS

```bash
set JAAS_OPTS=-Djava.security.auth.login.config=file:%~dp0../../config/kafka_jaas.conf
```

4. Modify the kafka-run-class.bat script to create java command with parameter %JAAS\_OPTS%

```bash
set COMMAND=%JAVA% %KAFKA_HEAP_OPTS% %KAFKA_JVM_PERFORMANCE_OPTS% %KAFKA_JMX_OPTS% %KAFKA_LOG4J_OPTS% %JAAS_OPTS% -cp "%CLASSPATH%" %KAFKA_OPTS% %*
```

5. Then start the zookeeper and kafka-server in different consoles, now the kafka server is setup correctly and running

```bash
zookeeper-server-start.bat .\config\zookeeper.properties
kafka-server-start.bat .\config\server.properties
```

6. Next we need to modify the consumer.properties/producer.properties to allow connecting to port 9092 and 9093

```bash
bootstrap.servers=localhost:9092, localhost:9093
```

7. Next we are going to make a test with the consumer and producer script with the plaintext port 9092, first let us create a topic 'gaming-events' (you don't have to create the topic for port 9093 again)

```bash
kafka-topics.bat --create --topic gaming-events --bootstrap-server localhost:9092
```

8. Then we start a consumer with the plaintext port 9092, the consumer is waiting message there :-)

```bash
kafka-console-consumer.bat --topic gaming-events --from-beginning --bootstrap-server localhost:9092
```

9. Then we start a producer with the unsecure port 9092, we can input some messages and verify the messages are received by consumer in the previous terminal

```bash
kafka-console-producer.bat --topic gaming-events --bootstrap-server localhost:9092
```

10. Next let's play with the secure port 9093, it is more interesting :-), the configuration is easy for the server, but it is a little bit confusing for the client scripts, we can config through the user/password through the .properties file or through the .conf file, but we always NEED to add the following settings in the consumer.properties/producer.properties

```bash
security.protocol=SASL_PLAINTEXT
sasl.mechanism=PLAIN
```

11. Then we will firstly config user/password by the .properties file, add the following content in the consumer.properties/producer.properties

```bash
sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="alice" password="alice-secret";
```

12. Next we start the consumer and producer as below, **DO NOT FORGET** **the option --consumer.config/--producer.config, the consumer.properties/producer.properties will not be read automatically**, this is the BIGGEST PIT that confused me a few days, I hate it. Now you are free to get the message flow from producer to consumer :-)

```bash
kafka-console-consumer.bat --topic gaming-events --from-beginning --bootstrap-server localhost:9093 --consumer.config .\config\consumer.properties
kafka-console-producer.bat --topic gaming-events --bootstrap-server localhost:9093 --producer.config .\config\producer.properties
```

13. Finally, let's do something more interesting, let us config the user/password through the .conf (of course you can name the file with any suffix as you want) config file, add the following content in the file 'kafka\_jaas.conf'

```bash
KafkaClient {
    org.apache.kafka.common.security.plain.PlainLoginModule required
    username="alice"
    password="alice-secret";
};
```

14. Then remove the following setting from the consumer.properties/producer.properties file

```bash
sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="alice"   password="alice-secret";
```

15. Next modify the kafka-console-producer.bat/kafka-console-consumer.bat by setting jvm

```bash
set JAAS_OPTS=-Djava.security.auth.login.config=file:%~dp0../../config/kafka_jaas.conf
```

16. Then we start the consumer and producer as usual, **DO NOT FORGET** the option --consumer.config/--producer.config as always, and the consumer/producer connect to the server with authentication :-)

```bash
kafka-console-consumer.bat --topic gaming-events --from-beginning --bootstrap-server localhost:9093 --consumer.config .\config\consumer.properties
kafka-console-producer.bat --topic gaming-events --bootstrap-server localhost:9093 --producer.config .\config\producer.properties
```