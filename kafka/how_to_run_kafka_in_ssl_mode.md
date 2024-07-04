# How to run kafka in SSL Mode?

**The authentication is different** between **SSL** mode and **SASL_SSL** mode, the **SSL** mode will use the **keystore** (holding the client's **private key**, client's **certificate signed by CA**) to authenticate. But **SASL_SSL** will use its own way to authenticate like  **user/pasword**, **oauthtoken** etc. For **SASL_SSL** mode please refer to the article [How to run kafka in SASL_SSL ](https://github.com/leiwang008/documents/blob/main/kafka/how_to_run_kafka_in_sasl_ssl_mode.md)

​

1. Generate the '**keystore**' and '**truststore**' on your kafka broker, please refer to the article [How to run kafka in SASL_SSL Mode](https://github.com/leiwang008/documents/blob/main/kafka/how_to_run_kafka_in_sasl_ssl_mode.md) 
2. Now let us config the kafka server.properties file as below, now you config the kafka in SSL mode on port 9093 

```bash
listeners=SSL://localhost:9093
advertised.listeners=SSL://localhost:9093
security.inter.broker.protocol=SSL
sasl.mechanism.inter.broker.protocol=PLAIN
sasl.enabled.mechanisms=PLAIN

# ssl configurations
ssl.keystore.location=/path_to/kafka.keystore
ssl.keystore.type=pkcs12
ssl.keystore.password=yourpass
ssl.key.password=yourpass
ssl.truststore.location=/path_to/kafka.truststore
ssl.truststore.type=pkcs12
ssl.truststore.password=yourpass
ssl.client.auth=required
```


- **Be careful with the store type settings**, we must set them as we generated the store in format '**pkcs12**'. If we don't sepcify them, the default type should be '**jks**' and you will meet error

```bash
ssl.keystore.type=pkcs12
ssl.truststore.type=pkcs12
```

- **Also be careful with the client auth setting 'ssl.client.auth'**, if we don't set this then only the broker will be verified by the client to see if the broker is really certified by a valid CA, and only **ssl.truststore.\*\*\*** settings will be needed by client (consumer/producer); If we set this field to "required", the broker will also verify the client is certified by a valid CA, and **ssl.keystore.\*\*\*** settings will also be needed by client.

```bash
ssl.client.auth=required
```

3. Then start the zookeeper and kafka-server in different consoles, now the kafka server is setup correctly and running

```bash
zookeeper-server-start.bat .\config\zookeeper.properties
kafka-server-start.bat .\config\server.properties
```

​

4. Next we need to modify the **consumer.properties/producer.properties** to allow connecting to port 9093 with protocol SSL, you can also copy the following content to a file 'client.properties' for kafka-topics.bat to use.

```bash
bootstrap.servers=localhost:9092, localhost:9093
sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="alice" password="alice-secret";
security.protocol=SASL_SSL
sasl.mechanism=PLAIN

#ssl configurations
ssl.truststore.location=/path_to/kafka.truststore
ssl.truststore.type=pkcs12
ssl.truststore.password=yourpass
#the following keystore setting are not needed if server didn't startup with 'ssl.client.auth=required'
ssl.keystore.location=/path_to/kafka.keystore
ssl.keystore.type=pkcs12
ssl.keystore.password=yourpass
```

5. Test the ssl connection with the following command

```bash
openssl s_client -connect localhost:9093 -tls1_2
```

if everything runs correctly, you should be able to get something as below

```bash
Connecting to 20.36.258.36
CONNECTED(00000194)
```

6. Create and List topic with port 9093 in SSL mode

```bash
kafka-topics.bat --create --topic gaming-events --bootstrap-server localhost:9093 --command-config ./config/client.properties
kafka-topics.bat --list --bootstrap-server localhost:9093 --command-config ./config/client.properties
```

7. Run Consumer with port 9093 in SSL mode

```bash
kafka-console-consumer.bat --topic gaming-events --from-beginning --bootstrap-server localhost:9093 --consumer.config ./config/consumer.properties
```

8. Run Producer with port 9093 in SSL mode

```bash
kafka-console-producer.bat --topic gaming-events --bootstrap-server localhost:9093 --producer.config ./config/producer.properties
```

​

​

​

​

​